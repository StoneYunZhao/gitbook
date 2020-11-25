# Meta

Meta 数据存储的是系统的内部信息，比如用户信息、数据库信息、RP 信息、shard 的元数据、CQ、subscription 等。相关的配置有：

```text
[meta]
dir = ""
retention-autocreate = true
logging-enabled = true
```

## 数据结构

Meta 数据格式是一种多层的结构，底层（通讯层）是通过 Protobuf 的方式描述的，对于单节点的 InfluxDB，通过 Protobuf 协议与磁盘交互。

DAO 层数据格式定义见 meta.proto，下面给出了最外层的数据定义，可见主要存储的是节点、数据库、用户、Shard 等信息。

```text
// services/meta/internal/meta.proto
message Data {
    required uint64 Term = 1;
    required uint64 Index = 2;
    required uint64 ClusterID = 3;

    repeated NodeInfo Nodes = 4;
    repeated DatabaseInfo Databases = 5;
    repeated UserInfo Users = 6;

    required uint64 MaxNodeID = 7;
    required uint64 MaxShardGroupID = 8;
    required uint64 MaxShardID = 9;

    // added for 0.10.0
    repeated NodeInfo DataNodes = 10;
    repeated NodeInfo MetaNodes = 11;
}

// COMMANDS 相关
...
```

业务层（DTO）数据格式大致与 DAO 类似，两者提供了相互转换的方法，即种 Struct 都有对应的 marshal 和 unmarshal 方法：

```go
// services/meta/data.go
type Data struct {
    Term      uint64 // associated raft term
    Index     uint64 // associated raft index
    ClusterID uint64
    Databases []DatabaseInfo
    Users     []UserInfo

    // adminUserExists provides a constant time mechanism for determining
    // if there is at least one admin user.
    adminUserExists bool

    MaxShardGroupID uint64
    MaxShardID      uint64
}

func (data *Data) marshal() *internal.Data {
    pb := &internal.Data{
        Term:      proto.Uint64(data.Term),
        Index:     proto.Uint64(data.Index),
        ClusterID: proto.Uint64(data.ClusterID),

        MaxShardGroupID: proto.Uint64(data.MaxShardGroupID),
        MaxShardID:      proto.Uint64(data.MaxShardID),

        // Need this for reverse compatibility
        MaxNodeID: proto.Uint64(0),
    }

    pb.Databases = make([]*internal.DatabaseInfo, len(data.Databases))
    for i := range data.Databases {
        pb.Databases[i] = data.Databases[i].marshal()
    }

    pb.Users = make([]*internal.UserInfo, len(data.Users))
    for i := range data.Users {
        pb.Users[i] = data.Users[i].marshal()
    }

    return pb
}

func (data *Data) unmarshal(pb *internal.Data) {
    data.Term = pb.GetTerm()
    data.Index = pb.GetIndex()
    data.ClusterID = pb.GetClusterID()

    data.MaxShardGroupID = pb.GetMaxShardGroupID()
    data.MaxShardID = pb.GetMaxShardID()

    data.Databases = make([]DatabaseInfo, len(pb.GetDatabases()))
    for i, x := range pb.GetDatabases() {
        data.Databases[i].unmarshal(x)
    }

    data.Users = make([]UserInfo, len(pb.GetUsers()))
    for i, x := range pb.GetUsers() {
        data.Users[i].unmarshal(x)
    }

    data.adminUserExists = data.hasAdminUser()
}
```

## Client

client 是 Meta 模块暴露出去的接口，外部对 Meta 数据的所有操作都通过 Client，Client 在服务启动的时候初始化：

```go
// services/meta/client.go
type Client struct {
    mu        sync.RWMutex
    closing   chan struct{}
    changed   chan struct{}
    cacheData *Data

    authCache map[string]authUser
}

func NewClient(config *Config) *Client {
    return &Client{
        cacheData: &Data{
            ClusterID: uint64(rand.Int63()),
            Index:     1,
        },
        closing:             make(chan struct{}),
        changed:             make(chan struct{}),
        authCache:           make(map[string]authUser),
    }
}
```

下面是系统初始化时，新建 Client 的调用栈：

```go
github.com/influxdata/influxdb/services/meta.NewClient at client.go:73
github.com/influxdata/influxdb/cmd/influxd/run.NewServer at server.go:176
github.com/influxdata/influxdb/cmd/influxd/run.(*Command).Run at command.go:142
main.(*Main).Run at main.go:81
main.main at main.go:45
```

Server 代表一个 InfluxDB 进程，里面包含一个 MetaClient 的 field，这个 MetaClient 会暴露给各个模块使用。

下面我们：加载数据 =&gt; 查询数据 =&gt; 更新数据 =&gt; 持久化数据 的逻辑来看 Client 是怎么实现的

### 加载数据

Server 初始化 MetaClient 后，立即调用 MetaClient 的 Open 方法，Open 方法会加载磁盘的上的 meta 目录，通过 protobuf 反序列化成 DAO 层的 data，然后再转换成 DTO 层的 data：

```go
// cmd/influxd/run/server.go
func NewServer(c *Config, buildInfo *BuildInfo) (*Server, error) {
  s := &Server{
        MetaClient: meta.NewClient(c.Meta),
    }

  if err := s.MetaClient.Open(); err != nil {
        return nil, err
    }
}

// services/meta/client.go
func (c *Client) Open() error {
    if err := c.Load(); err != nil {
        return err
    }
    return nil
}

func (c *Client) Load() error {
    file := filepath.Join(c.path, metaFile)

    f, err := os.Open(file)
    data, err := ioutil.ReadAll(f)

    if err := c.cacheData.UnmarshalBinary(data); err != nil {
        return err
    }
    return nil
}

func (data *Data) UnmarshalBinary(buf []byte) error {
    var pb internal.Data
    if err := proto.Unmarshal(buf, &pb); err != nil {
        return err
    }
    data.unmarshal(&pb) // 见数据结构一节的 unmarshal 实现
    return nil
}
```

### 查询数据

数据从磁盘加载进来后会提供给各个模块查询，Client 有很多查询方法，这里不一一列出了。查询逻辑比较简单，下面只给出一些例子：

```go
func (c *Client) Database(name string) *DatabaseInfo {
    for _, d := range c.cacheData.Databases {
        if d.Name == name {
            return &d
        }
    }
}

func (c *Client) Users() []UserInfo {
    users := c.cacheData.Users
    if users == nil {
        return []UserInfo{}
    }
    return users
}
```

### 更新数据

这里个更新数据包括删除 database、创建用户等各种操作。为了保证数据一致性，这里实现采用复制一份数据的方式，即每次更新数据时都会先复制一份，然后更新复制出来的数据，最后替换老的数据。

绝大部分更新操作都采用如下范式：

```go
// services/meta/client.go
func (c *Client) UpdateMethod(...) (...) {
  data := c.cacheData.Clone() // 复制一份数据

  // do update

  c.commit(data) // 替换老数据，同时也会持久化

  return ...
}

func (c *Client) commit(data *Data) error {
    data.Index++

  // 在更新内存数据之前，会先持久化到磁盘。持久化实现见下节
    if err := snapshot(c.path, data); err != nil {
        return err
    }

    c.cacheData = data

    close(c.changed)
    c.changed = make(chan struct{})

    return nil
}
```

比如创建 database、创建 shard group，见如下代码：

```go
// services/meta/client.go
func (c *Client) CreateDatabase(name string) (*DatabaseInfo, error) {
    data := c.cacheData.Clone()

    if db := data.Database(name); db != nil {
        return db, nil
    }

    if err := data.CreateDatabase(name); err != nil {
        return nil, err
    }

    // create default retention policy
    if c.retentionAutoCreate {
        rpi := DefaultRetentionPolicyInfo()
        if err := data.CreateRetentionPolicy(name, rpi, true); err != nil {
            return nil, err
        }
    }

    db := data.Database(name)

    if err := c.commit(data); err != nil {
        return nil, err
    }

    return db, nil
}

func (c *Client) CreateShardGroup(database, policy string, timestamp time.Time) (*ShardGroupInfo, error) {
    data := c.cacheData.Clone()
    if sg, _ := data.ShardGroupByTimestamp(database, policy, timestamp); sg != nil {
        return sg, nil
    }

    sgi, err := createShardGroup(data, database, policy, timestamp)
    if err != nil {
        return nil, err
    }

    if err := c.commit(data); err != nil {
        return nil, err
    }

    return sgi, nil
}
```

### 持久化数据

持久化操作在两种场景会触发：

* 各种更新操作，见上节。
* 在初始化 Client 的时候，若这个节点的 Raft Index 是 1，则会立即持久化。

持久化的逻辑也比较简单：

* 创建临时文件
* 通过 protobuf 协议把内存结构的数据转换成字节数组
* 字节数组写入文件
* 把临时文件更新为正式文件

```go
// services/meta/client.go
func snapshot(path string, data *Data) error {
    filename := filepath.Join(path, metaFile)
    tmpFile := filename + "tmp"

    f, err := os.Create(tmpFile)

    var d []byte
    if b, err := data.MarshalBinary(); err != nil {
        return err
    } else {
        d = b
    }

    if _, err := f.Write(d); err != nil {
        return err
    }

    if err = f.Sync(); err != nil {
        return err
    }

    //close file handle before renaming to support Windows
    if err = f.Close(); err != nil {
        return err
    }

    return file.RenameFile(tmpFile, filename)
}
```

