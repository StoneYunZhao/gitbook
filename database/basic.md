# Redis

## 持久化

提供了多种不同级别的持久化方式:一种是 **RDB**，另一种是 **AOF。**

**RDB 持久化**可以在指定的时间间隔内生成数据集的时间点快照（point-in-time snapshot）。

**AOF 持久化**记录服务器执行的所有写操作命令，并在服务器启动时，通过重新执行这些命令来还原数据集。 AOF 文件中的命令全部以 Redis 协议的格式来保存，新命令会被追加到文件的末尾。 Redis 还可以在后台对 AOF 文件进行重写（rewrite），使得 AOF 文件的体积不会超出保存数据集状态所需的实际大小。

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

