# Storage

[B 树](../../computer-science/algorithm/tree.md#b-shu)和 [B+ 树](../../computer-science/algorithm/tree.md#b-shu-1)的索引结构我们已经介绍，但是它们是怎么在磁盘上存储的呢？

记录是按照行存储的，但是读取的时候并不是以行为单位。在数据库中，无论读多少行，都是将这些行所在的页加载。也就是说，数据库磁盘存储的基本单位是页（Page）。

如下图，一个表空间包含一个或多个段，一个段包含一个或多个区，一个区包含了多个页，一个页有多行记录。

![](../../.gitbook/assets/image%20%28199%29.png)

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

![](../../.gitbook/assets/image%20%2854%29.png)

* 文件通用部分：**文件头**（File Header）和**文件尾**（File Tailer），类似包装盒，将页的内容包装起来，通过文件头和文件尾检验的方式来确保页传输是完整的。
  * **文件头**中有两个字段，FIL\_PAGE\_PREV 和 FIL\_PAGE\_NEXT，分别指向上一数据页和下一数据页，相当于双向链表。所以一个区中 64个页是逻辑上的连续，而不是物理上的。
  * 文件头给出 checksum ，到文件尾的时候整体运算出 checksum，两者一致才说明整个页传输是完整的。

![](../../.gitbook/assets/image%20%2831%29.png)

* 记录部分：**最大最小记录**（Infimum+supremum）、**用户记录**（User Records）、**空闲空间**（Free Space）占用了页结构的主要空间。当有新的记录插入时，会从空闲空间中进行分配存储。

![](../../.gitbook/assets/image%20%2871%29.png)

* 索引部分：**指页目录**（Page Directory），用户记录的目录，由于用户记录是顺序存储的，所以检索效率不高，有了页目录，就有二分查找的方式。如下图：

1. 一个数据页中所有记录分成几组，不包括已删除记录。
2. 第一组：最小记录，只包含一条；最后一组：包含最大记录，1-8条记录；其他组：4-8条记录。
3. 每个组的最后一条记录中的头信息会存储该组有多少条记录，n\_owned 字段。
4. 页目录存储每组最后一条记录的地址偏移量，每组的地址偏移量叫做槽（slot），每个槽指向不同组的最后一条记录。

![](../../.gitbook/assets/image%20%2845%29.png)

我们看看怎么进行二分查找，比如我们要查找主键为 9 的记录：

1. low = 0, high = 4, \(0 + 4\) / 2 = 2，2 对应的最大记录为 8。
2. 9 &gt; 8，所以 low = 2, high = 4, \(2 + 4\) / 2 = 3，3 对应的最大记录为 12。
3. 9 &lt; 12，所以在槽 3 中进行查找，遍历槽 3，找到 9。

#### B+ 树与页的关系

* 在 B+ 树中，每个节点都是一个页。
* 同一层上的节点通过数据页中文件头的两个指针形成一个双向链表。
* 非叶子节点有多个索引行，每个索引行存储索引键和指向下一层页面的数据页指针。
* 叶子节点，存储了关键字和行记录，在数据页内部的多条记录是一个单向链表。通过对页目录进行二分查找。

![](../../.gitbook/assets/image%20%28136%29.png)
