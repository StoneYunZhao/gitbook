# MySQL

* [SQL 语法](sql.md)
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
brew install mysql

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
```

## 调优

数据库调优的目的是让数据库运行更快，响应时间更短，吞吐量更大。

但是怎么反馈快呢？一般可从如下几点：

* 用户反馈
* 日志分析
* 服务器资源监控
* 数据库内部监控

数据库调优的思路一般有如下几个步骤：

* 选择合适的 DBMS
* 优化表设计
  * 尽量遵循三范式
  * 若分析查询较多，尤其是联表查询，可采用反范式，即空间换时间
  * 选择合适数据类型。数值型优于字符型；字符型长度尽量短；字符型若长度确定用 CHAR
* 优化逻辑查询，即通过改变 SQL 内容让 SQL 执行效率更高
* 优化物理查询，比如高效地建立索引与合理地使用索引
* 使用缓存
* 库优化，如控制表数量、主从、分库分表

