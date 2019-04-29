# Redis

## 安装

```bash
yum intall -y redis
brew install redis

redis-cli -h 172.16.1.18

redis-cli -h 10.129.60.199 -p 31919
10.129.60.199:31919> auth 123qwe!@#

keys abc*
```

## 持久化

提供了多种不同级别的持久化方式:一种是 **RDB**，另一种是 **AOF。**

**RDB 持久化**可以在指定的时间间隔内生成数据集的时间点快照（point-in-time snapshot）。对读写性能影响很小，因为主进程 fork 一个子进程。缺点是恢复时会丢失一定时间的数据

**AOF 持久化**记录服务器执行的所有写操作命令，并在服务器启动时，通过重新执行这些命令来还原数据集。 AOF 文件中的命令全部以 Redis 协议的格式来保存，新命令会被追加到文件的末尾。 Redis 还可以在后台对 AOF 文件进行重写（rewrite），使得 AOF 文件的体积不会超出保存数据集状态所需的实际大小。AOF可以更好的保护数据不丢失，一般AOF会每隔1秒，通过一个后台线程执行一次fsync操作（现代操作系统写文件不会直接写入磁盘，而会写入 os cache），最多丢失1秒钟的数据。

Redis 还可以**同时使用** AOF 持久化和 RDB 持久化。 在这种情况下， 当 Redis 重启时， 它会优先使用 AOF 文件来还原数据集， 因为 AOF 文件保存的数据集通常比 RDB 文件所保存的数据集更完整。

你甚至可以**关闭持久化**功能，让数据只在服务器运行时存在。

动态关闭AOF：

```text
redis-cli config set appendonly no
```

动态打开AOF：

```text
redis-cli config set appendonly yes
```

永久关闭AOF：

```text
sed -e '/appendonly/ s/^#*/#/' -i /etc/redis/redis.conf  
```

永久打开AOF：

```text
将appendonly yes设置在redis.conf中
```

## 与 Memcached 的区别

* Redis 比 Memcached 有更加丰富的数据结构与数据操作方式。
* Memcached 没有原生的集群模式，redis 支持cluster 模式。

## 线程模型

### 文件事件处理器

Redis 基于 **Reactor 模式**开发了网络事件处理器，叫做**文件事件处理器**（File event handler）。这个处理器是单线程的，所以说 Redis 是**单线程模型**。

* 文件事件处理器采用 **IO 多路复用**模型，同时监听多个 Socket。
* 若被监听的 Socket 准备好执行accept、read、write、close等操作的时，就会产生对应的文件事件，文件事件处理器就会调用套接字之前关联好的事件处理器来处理这些事件。

![](../.gitbook/assets/image%20%2850%29.png)

文件事件处理器的结构包含4个部分：

* 多个socket
* IO多路复用程序
* 文件事件分派器
* 事件处理器
  * 命令请求处理器
  * 命令回复处理器
  * 连接应答处理器
  * 复制处理器
  * 等等

多个文件事件可能会并发地出现， 但 I/O 多路复用程序总是会将所有产生事件的套接字都入队到一个**队列**里面， 然后以**有序**、**同步**、**每次一个**套接字的方式向文件事件分派器传送套接字，即当上一个套接字产生的事件被处理完毕之后（该套接字为事件所关联的事件处理器执行完毕）， I/O 多路复用程序才会继续向文件事件分派器传送下一个套接字。

![](../.gitbook/assets/image%20%2828%29.png)

### IO 多路复用程序

通过包装常见的 `select` 、 `epoll` 、 `evport` 和 `kqueue` 这些 I/O 多路复用函数库来实现的，Redis 会选择效率最高的实现方式。

![](../.gitbook/assets/image%20%2860%29.png)

### 文件事件

文件事件是对套接字操作的抽象， 每当一个套接字准备好执行连接accept、read、write、close等操作时， 就会产生一个文件事件。

* 当套接字变得可读时（客户端对套接字执行 `write` 操作，或者执行 `close` 操作）， 或者有新的可应答套接字出现时（客户端对服务器的监听套接字执行 `connect` 操作）， 套接字产生 `AE_READABLE` 事件。
* 当套接字变得可写时（客户端对套接字执行 `read` 操作）， 套接字产生 `AE_WRITABLE` 事件。

### 事件处理器

* 如果是客户端要连接redis，那么会为socket关联**连接应答处理器**。
* 如果是客户端要写数据到redis，那么会为socket关联**命令请求处理器**。
* 如果是客户端要从redis读数据，那么会为socket关联**命令回复处理器**。
* 当主服务器和从服务器进行复制操作时， 主从服务器都需要关联特别为复制功能编写的**复制处理器**。

### 单线程效率也很高

* 纯内存操作。
* 基于非阻塞的IO 多路复用机制。
* 避免了多线程的频繁上下文切换。

## 数据类型

* **String**：一个 Key 对应一个 Value。
* **Hash**：一个 Key 对应一个 Map，适合于存储对象。这样就不需要全部取出整个 JSON 对象、反序列化、修改、再序列化存储，可以直接修改某个字段。
* **List**：一个 Key 对应一个 List，可以 push、pop、LRange 等操作。
* **Set**：一个 Key 对应一个 Set，可以进行交集、并集、差集等操作。
* **Sorted Set**：zset，将 Set 中的元素带一个权重，比如存储全班同学成绩，可以快速拿出前几名。

## 过期删除策略

Reis 使用定时删除和惰性删除结合：

* **定期删除**：每隔一段时间（默认100ms）**随机**抽取一些 key，看是否过期，若过期则删除。
* **惰性删除**：当获取某个 Key 时，redis 会检查该 Key 是否过期，若过期则删除。

## 淘汰策略

由此可见，Redis 若有很多 Key 过期了没有被删除，也没有被查询而导致被删除，那么 Redis 的内存会越占越多。所以需要淘汰策略。

* **volatile-lru**：从已设置过期时间的数据集（server.db\[i\].expires）中挑选最近最少使用的数据淘汰。
* **volatile-ttl**：从已设置过期时间的数据集（server.db\[i\].expires）中挑选将要过期的数据淘汰。
* **volatile-random**：从已设置过期时间的数据集（server.db\[i\].expires）中任意选择数据淘汰。
* **allkeys-lru**：从数据集（server.db\[i\].dict）中挑选最近最少使用的数据淘汰，**最常用**。
* **allkeys-random**：从数据集（server.db\[i\].dict）中任意选择数据淘汰
* **no-enviction**（驱逐）：禁止驱逐数据，一般没人用。

## 主从模式

## Cluster 模式

节点间采用 gossip 协议。

与 gossip 协议对应的是集中式（Storm），即所有元数据集中在一个地方（如 zookeeper）。

## 分布式锁

### 普通模式

**获取锁**：

* `SET [key] [random] NX PX 30000`。
* NX 表示只有 key 不存在的时候才会设置成功。
* PX 30000 表示30秒后锁自动释放。
* 随机值的意义在于获取锁后，超过30秒才完成，然后删除锁，随机值用于删除正确的锁。

**释放锁**：

```lua
if redis.call('get', KEYS[1]) == ARGV[1] then 
    return redis.call('del', KEYS[1]) 
else 
    return 0 
end
```

### RedLock

遭到质疑。详情 Google。

