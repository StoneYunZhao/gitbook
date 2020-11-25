# Source Code - In-Memory Index

我们可以理解为 TSM 存储的是正排索引，而 Index 存储的是倒排索引。Index 有两种实现，In-Memory index 和 TSI。最初，InfluxDB 仅提供了 In-Memory 实现，后来为了适应大量短暂的 series 的场景，避免内存不足，实现了 TSI。若要使用 TSI，可以通过如下配置：

```text
 # default inmem
index-version = "tsi1"
```

在 InfluxDB 进程启动的时候会根据 TSM 文件构建出 In-Memory Index。有两个相关的配置：

```text
max-series-per-database = 1000000
max-values-per-tag = 100000
```

In-Memory Index 有两种：

* Index：每个 database 目录会创建一个 Index。
* ShardIndex：每个 shard 目录会创建一个 ShardIndex。

![](https://tech-proxy.bytedance.net/tos/images/1594731862497_22ce6938b2887440e307df15f6974224.png)

> 数据库文件目录的层级结构为：db =&gt; rp =&gt; shard

### 注册索引创建函数

系统启动的时候会向 tsdb 注册创建 Index 的函数，会注册两个函数，一个用于创建 Index，另一个用于创建 ShardIndex：

```go
// engine.go
var NewInmemIndex func(name string, sfile *SeriesFile) (interface{}, error)

// index.go
var newIndexFuncs = make(map[string]NewIndexFunc)

func RegisterIndex(name string, fn NewIndexFunc) {
    if _, ok := newIndexFuncs[name]; ok {
        panic("index already registered: " + name)
    }
    newIndexFuncs[name] = fn
}
```

```go
// inmem.go
func init() {
    tsdb.NewInmemIndex = func(...) ... { return NewIndex(name, sfile), nil }

    tsdb.RegisterIndex(IndexName, func(...) ... {
        return NewShardIndex(id, seriesIDSet, opt)
    })
}

func NewIndex(database string, sfile *tsdb.SeriesFile) *Index {
    index := &Index{...}
    ... // other initial work
    return index
}

func NewShardIndex(id uint64, seriesIDSet *tsdb.SeriesIDSet, opt tsdb.EngineOptions) tsdb.Index {
    return &ShardIndex{
    Index:        opt.InmemIndex.(*Index),
  }
}
```

### 创建索引实例

下面是主进程入口一直到 tsdb.\(\*Store\).Open 的流程。

```go
// stack
github.com/influxdata/influxdb/tsdb.(*Store).Open at store.go:221
github.com/influxdata/influxdb/cmd/influxd/run.(*Server).Open at server.go:442
github.com/influxdata/influxdb/cmd/influxd/run.(*Command).Run at command.go:149
main.(*Main).Run at main.go:81
main.main at main.go:45

// server.go
func (s *Server) Open() error {
  if err := s.TSDBStore.Open(); err != nil {
        return fmt.Errorf("open tsdb store: %s", err)
    }
}
```

Store 的 Open 操作会遍历 data 目录，为每个 db 创建一个 Index，为每个 Shard 创建一个 ShardIndex。

```go
// store.go
func (s *Store) Open() error {
  if err := s.loadShards(); err != nil {
        return err
    }
}

func (s *Store) loadShards() error {
  for _, db := range dbDirs { // 每个 db 目录
    idx, err := s.createIndexIfNotExists(db.Name()) // Index 是每个 db 创建一个

    for _, rp := range rpDirs { // 每个 rp 目录
      for _, sh := range shardDirs { // 每个 shard 目录
        go func(db, rp, sh string) {
          opt := s.EngineOptions
          opt.InmemIndex = idx //  通过 opt 把 db 级别的 Index 传递给 ShardIndex
          shard := NewShard(shardID, path, walPath, sfile, opt)
          err = shard.Open() // ShardIndex 是每个 Shard 创建一个
        }(db.Name(), rp.Name(), sh.Name())
      }
    }
  }
}

func (s *Store) createIndexIfNotExists(name string) (interface{}, error) {
    if idx := s.indexes[name]; idx != nil {
        return idx, nil
    }

    idx, err := NewInmemIndex(name, sfile) // 即注册的创建 Index 的函数，见注册 Index 一节
    return idx, nil
}

// shard.go
func (s *Shard) Open() error {
  if err := func() error {
    idx, err := NewIndex(s.id, s.database, ipath, seriesIDSet, s.sfile, s.options)
  }()
}

func NewShard(id uint64, path string, walPath string, sfile *SeriesFile, opt EngineOptions) *Shard {
  ...
}

// index.go
func NewIndex(...) (Index, error) {
  format := options.IndexVersion // 根据配置，拿到注册的创建 ShardIndex 的函数
  fn := newIndexFuncs[format]
  return fn(id, database, path, seriesIDSet, sfile, options), nil
}
```

ShardIndex 与 Index 是组合的关系，通过 opt 传递 Index，所以 ShardIndex 的大部分核心接口都是通过 Index 实现的。

### ShardIndex

ShardIndex 的方法分为四类：

* 删除 series，支持单个和批量。
* 创建 series
* SeriesIDSet 相关
* 仅仅是代理 Index 的方法

#### 数据结构

主要存储了两个字段：

* seriesIDSet：series Ids 的位图，通过 Roaring Bitmap 实现
* measurements：measurement name 到 series count 的 map

```go
type ShardIndex struct {
    id uint64
    *Index
    seriesIDSet *tsdb.SeriesIDSet 
    measurements map[string]int // measurement name -> series 的数量
    opt tsdb.EngineOptions
}
```

#### 删除 series

更新measurement 中 series 的数量。

```go
// 删除一个 series
func (idx *ShardIndex) DropSeries(seriesID uint64, key []byte, _ bool) error {
  if curr := idx.measurements[string(name)]; curr <= 1 {
      delete(idx.measurements, string(name))
  } else {
      idx.measurements[string(name)] = curr - 1
  }
}

// 删除多个，逻辑与上面类似
func (idx *ShardIndex) DropSeriesList(seriesIDs []uint64, keys [][]byte, _ bool) error {...}
```

#### 创建 Series

```go
func (idx *ShardIndex) CreateSeriesListIfNotExists(keys, names [][]byte, tagsSlice []models.Tags) error {
  keys, names, tagsSlice = idx.assignExistingSeries(...)
  if err := idx.Index.CreateSeriesListIfNotExists(...); err != nil {
        reason = err.Error()
        droppedKeys = append(droppedKeys, keys...)
    }
}

func (i *Index) assignExistingSeries(...) (...) {
    for j, key := range keys {
        if ss := i.series[string(key)]; ss == nil {
            keys[n] = keys[j]
            names[n] = names[j]
            tagsSlice[n] = tagsSlice[j]
            n++
        } else {
            if !seriesIDSet.Contains(ss.ID) {
                if !seriesIDSet.ContainsNoLock(ss.ID) {
                    seriesIDSet.AddNoLock(ss.ID)
                    measurements[string(names[j])]++
                }
            }
        }
    }
    return keys[:n], names[:n], tagsSlice[:n]
}
```

#### SeriesIDSet 相关

```go
func (idx *ShardIndex) SeriesN() int64 {
  return int64(idx.seriesIDSet.Cardinality())
}

func (idx *ShardIndex) SeriesIDSet() *tsdb.SeriesIDSet {
    return idx.seriesIDSet
}
```

#### 代理 Index 的方法

本身没有逻辑，都是代理 Index 的方法

```go
func (idx *ShardIndex) DropMeasurementIfSeriesNotExist(name []byte) (bool, error) {
    curr := idx.measurements[string(name)]
    if curr > 0 {
        return false, nil
    }
    _, err := idx.Index.DropMeasurementIfSeriesNotExist(name)
    return err == nil, err
}

func (idx *ShardIndex) InitializeSeries(keys, names [][]byte, tags []models.Tags) error {
    return idx.Index.CreateSeriesListIfNotExists(...)
}

func (idx *ShardIndex) CreateSeriesIfNotExists(key, name []byte, tags models.Tags) error {
    return idx.Index.CreateSeriesListIfNotExists(...)
}

func (idx *ShardIndex) TagSets(name []byte, opt query.IteratorOptions) ([]*query.TagSet, error) {
    return idx.Index.TagSets(...)
}
```

### Index

#### 数据结构

主要包含各种维度的索引，都是通过 map 实现：

* series： series key 到 series 的索引，series key = measurement + tag set
* measurements：measurement name 到 measurement 的索引

```go
type Index struct {
    mu sync.RWMutex

    database string
    sfile    *tsdb.SeriesFile
    fieldset *tsdb.MeasurementFieldSet

  measurements map[string]*measurement // key: measurement name
  series       map[string]*series // key: series key(measurement + tag set)

    seriesSketch, seriesTSSketch             estimator.Sketch
    measurementsSketch, measurementsTSSketch estimator.Sketch

    rebuildQueue sync.Mutex
}

// series 相关
type series struct {
    mu      sync.RWMutex
    deleted bool

    ID          uint64
    Measurement *measurement
    Key         string
    Tags        models.Tags
}

type Tags []Tag

type Tag struct {
    Key   []byte
    Value []byte
}

// measurement 相关
type measurement struct {
    Database  string
    Name      string `json:"name,omitempty"`
    NameBytes []byte 

    mu         sync.RWMutex
    fieldNames map[string]struct{}

  seriesByID          map[uint64]*series // key: series id  
  seriesByTagKeyValue map[string]*tagKeyValue // key: tag key

    sortedSeriesIDs seriesIDs 

    dirty bool
}

type tagKeyValue struct {
    mu      sync.RWMutex
  entries map[string]*tagKeyValueEntry // key: tag value
}

type tagKeyValueEntry struct {
  m map[uint64]struct{} // key: series id
    a seriesIDs  // 有序的 series ids
}

// 适合做排序、、交集、并集等操作
type seriesIDs []uint64 // element: series id
```

如下图，Index 是一个多级的 Map 结构：

![](https://tech-proxy.bytedance.net/tos/images/1594731898281_f0b411fa2fab8c06a8637f28787afa6c.png)

