# Storage

[B 树](../../computer-science/algorithm/tree.md#b-shu)和 [B+ 树](../../computer-science/algorithm/tree.md#b-shu-1)的索引结构我们已经介绍，但是它们是怎么在磁盘上存储的呢？

记录是按照行存储的，但是读取的时候并不是以行为单位。在数据库中，无论读多少行，都是将这些行所在的页加载。也就是说，数据库磁盘存储的基本单位是页（Page）。

如下图，一个表空间包含一个或多个段，一个段包含一个或多个区，一个区包含了多个页，一个页有多行记录。

![](../../.gitbook/assets/image%20%28194%29.png)

**表空间**（Tablespace）是一个逻辑容器，与段是 1:n 的关系。数据库由一个或多个表空间组成，从管理上可分为系统表空间、用户表空间、撤销表空间、临时表空间。查看表空间类型：

```sql
# ON 表示每张表会单独保存为一个 .ibd 文件
show variables like 'innodb_file_per_table';
```

**段**（Segment）与区是 1:n 的关系，段不要求区与区之间是连续的。当我们创建数据表、索引的时候，会创建相应的段，比如表段、索引段。

**区**（Extent）在 InnoDB 引擎中，一个区会分配 64 个连续的页，所以区的默认大小是 64 \* 16KB = 1MB。

**页**（Page）存储多行记录，在 InnoDB 中默认大小为 16KB。按照类型分的话，常见的有数据页（保存 B+树的节点）、系统页、Undo 页、事务数据页等。查看页的大小：

```sql
show variables like '%innodb_page_size%';
```

**数据页**包含 7 个部分，如下图：

![](../../.gitbook/assets/image%20%2852%29.png)

* 文件头（File Header）：

