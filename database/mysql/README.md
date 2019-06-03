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

数据定义语句（DDL）

```sql
# 获取表信息
decribe {tb_name};
decribe app;

# 增加列
ALTER TABLE {tb_name}
ADD [COLUMN] {col_name1} {col_definition1} [FIRST|AFTER existing_column1],
ADD [COLUMN] {col_name2} {col_definition2} [FIRST|AFTER existing_column2],...;

ALTER TABLE app ADD created_by varchar(32) NULL COMMENT '创建人';

# 删除列
ALTER TABLE {tb_name}
DROP COLUMN {col_name1},
DROP COLUMN {col_name2},...;

ALTER TABLE app DROP COLUMN created_by;
```

