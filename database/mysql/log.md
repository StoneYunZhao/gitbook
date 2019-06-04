# Log

### redo log

MySQL 如果每一次更新操作都写进磁盘，然后磁盘需要找到对应的记录，再更新，整个过程 IO成本很高。为了解决这个问题，就有了 WAL（Write-ahead Logging）技术。

即每当有一条记录需要更新的时候，InnoDB 会先把记录写到 redo log 里面，并更新内存，此时这个更新操作就可以返回了。InnoDB 会在合适的时候将 redo log 的内容写到磁盘，一般是系统空闲的时候。

redo log 的大小是固定的，可以配置。如下图，配置了4个文件，每个文件1GB。从头开始写，写到末尾就又回到开头循环写。

* **write pos**：当前写入位置，边写边往后移，写到3号文件末尾就回到0号文件开头。
* **check point**：当前擦除位置，也是往后移并循环。

write pos 和 check point 之间是空的部分，可以用来记录新的操作。若 write pos 追上了 check point，InnoDB 就会停止更新操作，先把 redo log 的内容写入一部分到磁盘，即使得 check point 往后移动一段距离。

`innodb_flush_log_at_trx_commit`设置为1时，表示每次事务的 redo log都直接持久化到磁盘。

![](../../.gitbook/assets/image%20%283%29.png)

### binlog

binlog 属于 Server 层的。

binlog 有两种模式，**statement 模式**记录的是 sql 语句；**row 模式**记录行的内容，更新前后两条记录。

`sync_binlog`设置为1时，表示每次事务的 binlog 都持久化到磁盘。

### 异同点

* redo log 是 InnoDB 特有的，binlog 是属于 Server 层，所有的存储引擎都可以使用。
* redo log 是物理日志，记录在某个数据页上做了什么修改；binlog 是逻辑日志，记录的是更新语句的原始逻辑，比如“ID=2 这一行的 c 字段加1”。
* redo log 是循环写的，空间固定会用完；binlog 是追加写入，binlog 文件到了一定大小后切换到下一个，并不会覆盖以前的日志。

### 两阶段提交

```sql
update T set c=c+1 where ID=2;
```

对于上面的 SQL 语句，大致执行流程如下：1

1. 执行器找到 ID=2 这一行，若数据在内存中，则直接返回；否则先从磁盘读入内存。
2. 执行器拿到数据，把 c 值加1，再调用存储引擎写入数据。
3. 存储引擎将数据写入内存，并将update 操作写入 redo log，此时 redo log 处于 prepare 状态。
4. 执行器生产update 操作的 binlog，并把 binlog 写入磁盘。
5. 执行器调用存储引擎提交事务接口，把 redo log 的状态改为 commit。

![](../../.gitbook/assets/image%20%2896%29.png)

上面最后三步就是**两阶段提交**。

