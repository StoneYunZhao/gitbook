# MySQL

## 安装

```bash
# 仅安装客户端
yum install mysql

# 仅安装服务端
yum install mysql-server
```

## 基本命令

```bash
# 连接 MySQL
mysql -u {username} -p {password} \
    -h {remote server ip or name} -P {port} \
    -D {DB name}
    
mysql -u pcopadm -h 127.0.0.1 -P 3306 -p

drop schema XXX;
```

