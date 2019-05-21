# Indexing

索引的目的是为了提高数据的查询效率。

## 1. 常见索引数据结构

### 1.1 Hash 表

本质是**数组+链表**的数据结构。

**缺点**：做区间查询速度很慢。

**适用场景**：只有等值查询的情况。比如 Memcached 、Lucene 等。

### 1.2 有序数组

等值查询和范围查询都很快。等值查询用用二分查找，时间复杂度`O(log(N))`。范围查询先用等值查询找到第一个，然后往后遍历。

**缺点**：插入数据效率很低。

**适用场景**：静态数据，比如 2015年上海市的人口信息。

### 1.3 [平衡二叉查找树](../../computer-science/algorithm/tree.md#ping-heng-er-cha-cha-zhao-shu)

### 1.4 B+ 树

二叉树的搜索效率最高，但是很多数据库不使用二叉树，比如 [MySQL](indexing.md#2-mysql-suo-yin)，原因是索引不止在内存中，还要写到磁盘上。为了减少磁盘的读取次数，所以应该使用 N 叉树，以 InnoDB 为例，N 大约为 1200。

假设有如下表和数据，则 InnoDB 的索引结构为下图：

```sql
mysql> create table T(
id int primary key, 
k int not null, 
name varchar(16),
index (k))engine=InnoDB;

(100,1) (200,2) (300,3) (500,5) (600,6)
```

![](../../.gitbook/assets/image%20%2869%29.png)

* **主键索引**：又叫聚簇索引（clustered index），叶子节点存整行数据。
* **非主键索引**：又叫耳机索引（secondary index），叶子节点存主键的值。

若使用`select * from T where k=5`，则先搜索 k 索引树，得到 ID 再去主键索引树搜索，这称为**回表**。

#### 页分裂

当插入数据时，比如插入 700，则只需在后面追加一条记录。若插入 400，需要逻辑上移动后面的数据。若插入的页已经满了，B+ 树会申请一个新的页，然后挪动部分数据过去。

#### 页合并

若相邻两个页由于删除数据，空间利用率很低，则会把数据页合并。

### 1.5 跳表

### 1.6 LSM 树

## 2. MySQL 索引

### 2.1 自增主键

自增主键指插入记录时，若不指定主键值，则会获取当前主键值并加 1 作为下一条主键的值。通过 `NOT NULL PRIMARY KEY AUTO_INCREMENT`定义

**优点**：插入数据是递增的，主键索引不会发生页分裂。且整型占用空间少，二级索引占用的空间也少。

### 2.2 索引重建

索引可能因为删除、页分裂等情况，导致数据空洞，重建索引会把数据按照顺序插入，页面利用率就高。

若需要重建索引，下面方法是否可行？

```sql
alter table T drop index k;
alter table T add index(k);

alter table T drop primary key;
alter table T add primary key(id);
```

重建索引 k 是合理的。但是重建主键索引不合理，删除、创建主键都会重建整个表，所以前面的重建 k 就白做了。可以使用`alter table T engine=InnoDB`替代。

### 2.3 索引覆盖

若查询需要的结果已经包含在二级索引里面，不需要回表，称为索引覆盖。是一个常用的性能优化手段。

### 2.4 最左匹配

最左匹配可以是联合索引的最左边的 N 个字段，也可以是字符串索引的最左 M 个字符。

联合索引的顺序考虑：

* 如果能够通过调整顺序，减少一个索引，那么应该使用这个顺序。比如已经有 \(a,b\) 索引，就不需要单独的 a 索引了。
* 空间原则，比如 name 和 age 两个字段，name 长度大于 age，那么应该创建 \(name,age\) 和 \(age\)，而不是 \(age,name\) 和 \(name\)。

### 2.5 索引下推

MySQL 5.6 引入索引下推优化（index condition pushdown），可以在索引遍历过程中，对索引中包含的字段先做判断，直接过滤掉不满足条件的记录。比如下面语句：

```sql
mysql> select * from tuser where name like '张 %' and age=10 and ismale=1;
```

在 5.6 之前需要回表四次：

![](../../.gitbook/assets/image%20%2834%29.png)

在 5.6 仅需回表两次：

![](../../.gitbook/assets/image%20%2857%29.png)

