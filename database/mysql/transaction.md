# Transaction

## 概念

事务就是指一组数据库操作，要么全部成功，要么全部失败。

### ACID

* Atomicity：原子性，进行数据处理操作的基本单位，不可分割。
* Consistency：一致性，数据库在进行事务操作后，会由原来的一致状态，变成另外一种一致状态。一致性一般由**业务定义**的。
* Isolation：隔离性，事务不受其它事务影响。
* Durability：持久性，事务提交之后对数据的修改是持久性的。

### 事务带来的问题

当数据库有多个事务同时执行的时候，可能会出现以下问题：

* **脏读（dirty read）**：一个事务读取了另一个事务未提交的数据。
* **不可重复读（non-repeatalble read）**：一个事务由于另一个事务的提交导致前后两次读到的**数据不一致**。
* **幻读（phantom read）**：一个事务由于另一个事务的提交导致前后两次读到的**数据行数不一致**。

### 隔离级别

隔离级别越高，效率越低，所以我们需要找一个平衡点。

* **read uncommitted**：一个事务还没提交，它的变更就能被别的事务看到。存在脏读、不可重复读、虚读的问题。
* **read committed**：一个事务被提交之后，它的变更才能被别的事务看到。存在不可重复读、虚读的问题。Oracle 的默认级别。
* **repeatable read**：一个事务在执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。存在虚读问题。MySQL 的默认级别。
* **serializable**：后访问的数据必须等前一个事务执行完成才能继续执行，是最高事务级别。

假设有如下表和数据，并且有两个事务 A、B：

```sql
mysql> create table T(c int) engine=InnoDB;
insert into T(c) values(1);
```

| 事务 A | 事务 B |
| :---: | :---: |
| 启动事务 A | 启动事务 B |
| 查询得到 1 |   |
|   | 查询得到 1 |
|   | 将 1 改为 2 |
| 查询得到 V1 |   |
|   | 提交事务 B |
| 查询得到 V2 |   |
| 提交事务 A |   |
| 查询得到 V3 |   |

那么在不同隔离级别下，各 V 的值为：

| 隔离级别 | V1 | V2 | V3 |
| :---: | :---: | :---: | :---: |
| 读未提交 |  2 | 2 | 2 |
| 读提交 | 1 | 2 | 2 |
| 可重复读 | 1 | 1 | 2 |
| 串行化 | 1 | 1 | 2 |

{% hint style="info" %}
可重复读和串行化所看到的结果虽然一样，但是执行方式不一样。串行化的情况下，当事务 B 在修改数据时，会被阻塞，直到事务 A 提交，事务 B 才能继续执行。
{% endhint %}

## 语法

官方文档，[Transactional and Locking Statements](https://dev.mysql.com/doc/refman/8.0/en/sql-syntax-transactions.html)。

* START TRANSACTION / BEGIN：显示开启一个事务。
* COMMIT：提交事务。
* ROLLBACK / ROLLBACK TO \[SAVEPOINT\]：回滚，或回滚到某个保存点。
* SAVEPOINT：创建保存点。
* RELEASE SAVEPOINT：删除保存点。
* SET TRANSACTION：设置隔离级别。
* SET @@completion\_type
  * 0：默认，执行 COMMIT 会提交事务，执行下一个事务时，需要用 START TRANSACTION / BEGIN 来开启。
  * 1：提交事务后，会自动开启一个相同隔离级别的事务。
  * 2：提交事务后，会自动与服务器断开连接。
* SET autocommit
  * 0：显示启动事务，关闭自动提交，比如执行一个 select 语句，事务就启动了，并且不会自动提交，事务持续直到执行 commit 或 rollback 或断开连接。
  * 1：隐式启动事务。

autocommit 与 START TRANSACTION：

* 当 autocommit=0，不管有没有 START TRANSACTION，COMMIT 才会生效，ROLLBACK 会回滚。
* 当 autocommit=1，没有 START TRANSACTION 或 BEGIN，COMMIT 和 ROLLBACK 是无用的。
* 不管 autocommit 是什么值，只要有 START TRANSACTION 或 BEGIN，意味着开启一个显示事务，必须要 COMMIT 才会生效，ROLLBACK 才会回滚。

### 案例

```sql
CREATE TABLE test(name varchar(255), PRIMARY KEY (name)) ENGINE=InnoDB; 
BEGIN; 
INSERT INTO test SELECT '关⽻'; 
COMMIT; 
BEGIN; 
INSERT INTO test SELECT '张⻜'; 
INSERT INTO test SELECT '张⻜'; 
COMMIT;
```

上面 sql 执行完后，数据表中就一行\`关羽\`，因为第二个事务由于主键冲突会执行失败，自动回滚。

```sql
CREATE TABLE test(name varchar(255), PRIMARY KEY (name)) ENGINE=InnoDB; 
BEGIN; 
INSERT INTO test SELECT '关⽻'; 
COMMIT; 
INSERT INTO test SELECT '张⻜'; 
INSERT INTO test SELECT '张⻜'; 
ROLLBACK;
```

上面 sql 执行完后，表中有两行数据。插入\`张飞\`时没有显式开启事务，那么每一行 sql 都自动称为一个事务。

```sql
CREATE TABLE test(name varchar(255), PRIMARY KEY (name)) ENGINE=InnoDB; 
SET @@completion_type = 1; 
BEGIN; 
INSERT INTO test SELECT '关⽻'; 
COMMIT; 
INSERT INTO test SELECT '张⻜'; 
INSERT INTO test SELECT '张⻜'; 
ROLLBACK;
```

上面 sql 执行完后，表中只有一行数据\`关羽\`。

## MySQL 事务

MyISAM 不支持事务，InnoDB 支持事务。

默认级别是可重复读，可以配置启动参数`transaction-isolation`。

```sql
# 查询超过 60s 的事务
select *
from information_schema.innodb_trx
where TIME_TO_SEC(timediff(now(), trx_started)) > 60;
```

### 实现原理

* 读未提交直接返回记录上的最新值。
* 读提交在每个 SQL 语句开始的时候创建视图。
* 可重复读在事务启动的时候创建视图。
* 串行化通过加锁来避免并行访问。

**以可重复读为例**，分析事务的原理。

MySQL 的每个更新操作都会记录一条回滚操作。如下图，一个值按照顺序由 1 被改成了 2、3、4。不同时刻启动的事务有不同的 read-view，即同一条记录可以存在多个版本，即**多版本并发控制（MVCC）**。

![](../../.gitbook/assets/image%20%28159%29.png)

**回滚日志的删除**：系统会判断，当没有事务在需要用到这些回滚日志，即没有比这个回滚日志更早的 read-view 时，回滚日志会被删除。

{% hint style="info" %}
所以尽量不要用长事务，因为会存在很老的视图，回滚日志就不能删除。
{% endhint %}

