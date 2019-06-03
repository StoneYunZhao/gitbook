# MySQL

* [架构](architecture.md)
* [日志](log.md)
* [事务](transaction.md)
* [索引](indexing.md)
* [锁](lock.md)

## 基本使用

安装

```bash
# 仅安装客户端
yum install mysql

# 仅安装服务端
yum install mysql-server
```

Docker 启动

```bash
docker pull mysql:8.0.16

docker run -d --name mysql \
-p 3306:3306 \
-e MYSQL_ROOT_PASSWORD=123qwe \
-v /Users/zhaoyun/Downloads/mysql:/var/lib/mysql \
mysql:8.0.16
```

基本命令

```bash
# 连接 MySQL
mysql -u {username} -p {password} \
    -h {remote server ip or name} -P {port} \
    -D {DB name}
    
mysql -u pcopadm -h 127.0.0.1 -P 3306 -p

drop schema XXX;
```

