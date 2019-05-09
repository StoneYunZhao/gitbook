# Lock

数据库锁的初衷是处理并发问题。

## 全局锁

全局锁就是对整个数据库实例加锁。

适用场景：全库做逻辑备份。比如一个库的某几张表有业务逻辑关系，所以需要整个库的备份是在同一个时间点。

缺点：

* 若在主库上加锁，则业务基本需要停止。
* 若在从库上加锁，则从库不能执行主库同步过来的 binlog。

### FTWRL

`Flush table with read lock`，让整个库处于只读状态，执行命令后，以下语句会被阻塞：

* 数据跟新语句（DML），即数据的增删改。
* 数据定义语句（DDL），比如创建表、修改表结构。
* 更新类事务的提交语句。

### mysqldump

借鉴[可重复读的隔离级别实现](transaction.md#shi-xian-yuan-li)，即 MVCC，我们也可以使用 MVCC 的方式来做全库备份，这样就不会阻塞写操作了。

官方有个自带工具 mysqldump，当使用参数`--single-transaction`时，导数据时会启动一个事务，拿到全库一致性的视图，由于有 MVCC 的支持，这个过程数据时可以正常更新的。

**缺点**：需要引擎支持这个隔离级别，比如 MyISAM 就不支持。所以此方法只适用于所有表使用的是支持事务引擎的库。

若全部是 InnoDB 引擎的库，建议使用此方法做全库备份。

### global 参数

使用`set global readonly=true`也可以使全库只读。

但是不建议使用，因为使用 FTWRL 后客户端断开连接，MySQL 会自动释放这个全局锁，整个库可以恢复到正常跟新；但是使用 global 参数，就算客户端发生异常，数据库也会保持 readonly 状态。

## 表级锁

### 表锁

`lock tables ... read/write`，可以用`unlock tables`主动释放，也可以在客户端断开时自动释放。

* 线程 A 执行`lock tables t1 read`，其它线程写 t1 会被阻塞，线程 A 不能写 t1。
* 线程 A 执行`lock tables t2 write`，其它线程 读写 t1 会被阻塞。

{% hint style="danger" %}
lock tables 也会限制本线程的操作对象，上面两种情况，线程 A 不能访问其它表。
{% endhint %}

对于不支持细粒度锁的引擎，一般用表锁。但是 InnoDB 支持行锁，很少用`lock tables`。

### 元数据锁

MDL（metadata lock），MySQL 5.5 引入，不需要显示使用，在访问一个表的时候会被自动加上。

* 对一个表做增删改查操作时，加 MDL 读锁；读锁不互斥。
* 对一个表做结构变更时，加 MDL 写锁；写锁与读锁、写锁都互斥。

{% hint style="info" %}
在alter table 的时候设定等待时间，因为如果长时间拿不到 MDL 写锁，也不要阻塞后面的业务。MariaDB 合并了 AliSQL 的这个功能，这两个分支都支持 DDL NOWAIT/WAIT n 的语法：

```sql
ALTER TABLE tbl_name NOWAIT add column ...
ALTER TABLE tbl_name WAIT N add column ... 
```
{% endhint %}

## 行锁

