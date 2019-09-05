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

select version();

# 显示所有引擎
show engines;
```

分析 SQL 执行时间

```bash
# 查看是否开启，关闭(0)，开启(1)
select @@profiling;

# 开启
set profiling = 1;

# 执行任意语句
select * from test;

# 查看当前会话所产生的 profiles
show profiles;

+----------+------------+--------------------+
| Query_ID | Duration   | Query              |
+----------+------------+--------------------+
|        1 | 0.00185800 | select @@profiling |
|        2 | 0.00108000 | select * from test |
+----------+------------+--------------------+
2 rows in set, 1 warning (0.00 sec)

# 获取上一次查询的详细执行时间
show profile;

# 查询指定 queryId 的详细执行时间
show profile for query 2;

+--------------------------------+----------+
| Status                         | Duration |
+--------------------------------+----------+
| starting                       | 0.000112 |
| Executing hook on transaction  | 0.000022 |
| starting                       | 0.000023 |
| checking permissions           | 0.000021 |
| Opening tables                 | 0.000065 |
| init                           | 0.000022 |
| System lock                    | 0.000046 |
| optimizing                     | 0.000018 |
| statistics                     | 0.000032 |
| preparing                      | 0.000033 |
| executing                      | 0.000056 |
| end                            | 0.000018 |
| query end                      | 0.000016 |
| waiting for handler commit     | 0.000036 |
| closing tables                 | 0.000022 |
| freeing items                  | 0.000293 |
| cleaning up                    | 0.000249 |
+--------------------------------+----------+
17 rows in set, 1 warning (0.01 sec)
```

