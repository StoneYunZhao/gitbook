# SQL

自从 SQL 加入 TIOBE 就一直保持在 Top 10。SQL 从诞生到现在，变化很少。SQL 有两个标准 SQL92 和 SQL99。

可以把 SQL 语言按照功能分为四种：

* **DDL\(Data Definition Language\)**：创建、删除、修改数据库和表结构。
* **DML\(Data Manipulation Language\)**：增加、删除、修改数据表中的记录。
* **DCL\(Data Control Language\)**：定义访问权限和安全级别。
* **DQL\(Data Query Language\)**：查询想要的记录。

书写 SQL 语句我们遵循如下规范：

* 表名、表别名、字段名、字段别名等小写；
* SQL 保留字、函数名、绑定变量等大写；

主流的 DBMS（数据库管理系统）有 Oracle、MySQL、SQL Server、PostgreSQL、DB2、MongoDB 等。

* DBMS（Database Management System）：可以对多个数据库进行管理，DBMS = 多个 DB + 管理程序。
* DB（Database）：数据库，可理解为多张数据表。
* DBS（Database System）：数据库系统，包括数据库、数据库管理系统、数据管理人员（DBA）等。
* RDBMS，关系型数据库。

我们采用 E-R 图（Entity Relationship Diagram）来设计数据关系， 这个图有 3 个要素：实体、属性、关系。

## DDL

官方文档，[Data Definition Statements](https://dev.mysql.com/doc/refman/8.0/en/sql-syntax-data-definition.html)。

常用的有增、删、改，分别对应 CREATE、DROP、ALTER。在执行 DDL 时，不需要 COMMIT 就能完成执行。

```sql
# 数据库
CREATE DATABASE XXX;
DROP DATABASE XXX;
DROP SCHEMA XXX;

# 创建数据表
# 数据和字段都是用了反引号，以防和保留字相同
# int(11)，其中 11 代表显示长度，与数值范围无关
# varchar(255)，其中 255 代表最大长度
# player_name 的字符集是 uft8，排序规则是 utf8_general_ci，表示大小写不敏感
DROP TABLE IF EXISTS `player`; 
CREATE TABLE `player` ( 
`player_id` int(11) NOT NULL AUTO_INCREMENT, 
`team_id` int(11) NOT NULL, 
`player_name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL, 
`height` float(3, 2) NULL DEFAULT 0.00, 
PRIMARY KEY (`player_id`) USING BTREE, 
UNIQUE INDEX `player_name`(`player_name`) USING BTREE 
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

# 增加列
ALTER TABLE {tb_name}
ADD [COLUMN] {col_name1} {col_definition1} [FIRST|AFTER existing_column1],
ADD [COLUMN] {col_name2} {col_definition2} [FIRST|AFTER existing_column2],...;

ALTER TABLE app ADD created_by varchar(32) NULL COMMENT '创建人';

# 删除列
ALTER TABLE {tb_name}
DROP COLUMN {col_name1},
DROP COLUMN {col_name2},...;

# 重命名列
ALTER TABLE XXX RENAME COLUMN age to player_age;

# 修改列数据类型
ALTER TABLE XXX MODIFY (player_age float(3,1));
```

### 数据表约束

* 主键约束：不能重复，不能为空，即 UNIQUE + NOT NULL；
* 外键约束：一个表的外键对应另一个表的主键；
* 唯一性约束：表明字段在表中的数值是唯一的；
* NOT NULL 约束：不能为空；
* DEFAULT：
* CHECK：用来检查特定字段取值范围的有效性；

### 数据表的设计原则：

* 数据表越少越好；
* 数据表中的字段个数越少越好；
* 联合主键的字段数越少越好；
* 主键的利用率越高越好；

## DQL

官方文档，[SELECT Syntax](https://dev.mysql.com/doc/refman/8.0/en/select.html)。

```sql
# 查询某些列
SELECT name, create_time FROM tag;

# 查询所有列
SELECT * from tag;

# 取别名
SELECT name AS n, create_time AS ct FROM tag;

# 查询常数列，这一列的取值是我们指定的，不是从数据库中动态获取，适合整合不同的数据源
SELECT '标签' AS `type`, name FROM tag;

# 去除重复行
# DISTINCT 需要放到所有列名前
# DISTINCT 是对后面所有列的组合进行去重
SELECT DISTINCT name FROM tag;

# 排序
# ORDER BY 后面可以有多个列
# ORDER BY 后面可以著名排序规则，ASC(default)、DESC，
# ORDER BY 后面的列可以不在选择列里面
# ORDER BY 通常位于 SELECT 语句的最后
SELECT name FROM tag ORDER BY create_time;

# 返回结果数量
SELECT name FROM tag LIMIT 5;
```

### WHERE 子句

官方文档，[Expressions](https://dev.mysql.com/doc/refman/8.0/en/expressions.html)、[Functions and Operators](https://dev.mysql.com/doc/refman/8.0/en/functions.html)。

比较运算符有：`= | >= | > | <= | < | <> | !=`。

```sql
SELECT * FROM tag WHERE id > 10;
SELECT * FROM tag WHERE id BETWEEN 10 AND 20;
SELECT * FROM tag WHERE name IS NULL;
```

逻辑运算符有：`AND OR IN NOT`。

通配符：`LIKE`。

### GROUP BY

我们可以对数据进行分组，并使用聚集函数统计每组数据的值。

```sql
SELECT COUNT(*) AS num, role_main, role_assist FROM heros GROUP BY role_main, role_assist ORDER BY num DESC;
```

### HAVING

当我们使用 GROUP BY 创建分组之后，可能需要对分组数据进行过滤，此时就需要使用 HAVING。WHERE 是作用于数据行，而 HAVING 是作用于分组。

```sql
SELECT COUNT(*) as num, role_main, role_assist
FROM heros
WHERE hp_max > 6000
GROUP BY role_main, role_assist
HAVING num > 5
ORDER BY num;
```

> The SQL standard requires that `HAVING` must reference only columns in the `GROUP BY` clause or columns used in aggregate functions. However, MySQL supports an extension to this behavior, and permits `HAVING` to refer to columns in the `SELECT` list and columns in outer subqueries as well.

SQL 中关键字是有顺序的：

```sql
SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY ...
```

### Subquery

官方文档，[Subquery Syntax](https://dev.mysql.com/doc/refman/8.0/en/subqueries.html)。

**非关联子查询**：子查询只执行一次，查询的数据结果作为主查询的条件。如：

```sql
# 查询最高身高的球员，查询最高身高只执行一次。
SELECT player_name, height
FROM player
WHERE height = (SELECT max(height) FROM player);
```

**关联子查询**：子查询需要执行多次，即采用循环的方式，先从外面开始，每次都传入子查询，最后将结果反馈给外部。如：

```sql
# 查询每个球队中大于平均身高的球员，查询球队的平均身高需要执行多次。
SELECT player_name, height, team_id
FROM player AS a
WHERE height > (SELECT avg(height) FROM player AS b WHERE a.team_id = b.team_id);
```

子查询还能作为主查询的计算字段：

```sql
# 查询每个球队的球员数量
SELECT team_name, 
(SELECT count(*) FROM player WHERE player.team_id = team.team_id) 
AS player_num
FROM team;
```

### EXISTS、NOT EXISTS

官方文档：[Subqueries with EXISTS or NOT EXISTS](https://dev.mysql.com/doc/refman/8.0/en/exists-and-not-exists-subqueries.html)。

如果子查询返回任何行，则 EXISTS 为 true。

```sql
# 查询有出场纪录的球员
SELECT player_id, team_id, player_name
FROM player
WHERE EXISTS(SELECT player_id FROM player_score WHERE player.playe_id = player_score.player_id);
```

### ANY、IN、SOME、ALL

官方文档：[Subqueries with ANY, IN, or SOME](https://dev.mysql.com/doc/refman/8.0/en/any-in-some-subqueries.html)，[Subqueries with ALL](https://dev.mysql.com/doc/refman/8.0/en/all-subqueries.html)。语法：

```sql
operand comparison_operator ANY (subquery)
operand IN (subquery)
operand comparison_operator SOME (subquery)
operand comparison_operator ALL (subquery)

comparison_operator:
=  >  <  >=  <=  <>  !=
```

* IN: 是否在集合中。
* ANY: 与子查询返回的任意一个值比较为真，则整体为真。
* ALL: 与子查询返回的所有值比较。
* SOME: 是 ANY 的别名，一般用 ANY。

如上文 EXISTS 查询有出场纪录的球员也可写成：

```sql
SELECT player_id, team_id, player_name
FROM player
WHERE player_id IN (SELECT player_id FROM player_score);
```

即 EXISTS 和 IN 的查询结果可等价：

```sql
SELECT * FROM A WHERE cc IN (SELECT cc FROM B);
SELECT * FROM A WHERE EXISTS (SELECT cc FROM B WHERE B.cc=A.cc)
```

{% hint style="info" %}
查询结果等价，但是**查询效率不等价**。在上面的例子中，假设 cc 列建立的索引。若 A 表较大，那么 IN 的效率较高；反之，EXISTS 效率较高。
{% endhint %}

```sql
# 查询比(球队 1002 中任意一个球员身高)高的球员
SELECT player_id, player_name, height
FROM player
WHERE height > ANY (SELECT height FROM player WHERE team_id = 1002);

# 查询比(球队 1002 中所有球员身高)都高的球员
SELECT player_id, player_name, height
FROM player
WHERE height > ALL (SELECT height FROM player WHERE team_id = 1002);
```

### JOIN

上文提到 SQL 常用的两个标准是 SQL92 和 SQL99，两个标准的连接查询语法也不一样。

#### CROSS JOIN

实际上就是求两个表的笛卡尔积，即行数 = A 行数 \* B 行数。

```sql
# SQL99
SELECT * FROM player CROSS JOIN team;
SELECT * FROM t1 CROSS JOIN t2 CROSS JOIN t3;

# SQL92
SELECT * FROM player, team;
```

#### NATURAL JOIN

帮你自动查询两张表中所有相同的字段，然后进行等值连接。

```sql
# SQL99
SELECT player_id, team_id, player_name, height, team_name
FROM player
       NATURAL JOIN team;

# SQL92
SELECT player_id, a.team_id, player_name, height, team_name
FROM player AS a,
     team AS b
WHERE a.team_id = b.team_id;
```

{% hint style="info" %}
若表使用了别名，那么查询字段中只能使用别名，不能使用原有的表名。
{% endhint %}

#### ON

用来指定连接条件。

```sql
# SQL99
SELECT player_id, player.team_id, player_name, height, team_name
FROM player
       JOIN team ON player.team_id = team.team_id;
       
SELECT p.player_name, p.height, h.height_level
FROM player as p
       JOIN height_grades as h ON height BETWEEN h.height_lowest AND h.height_highest;
    
# SQL92   
SELECT p.player_name, p.height, h.height_level
FROM player AS p,
     height_grades AS h
WHERE p.height BETWEEN h.height_lowest AND h.height_highest;
```

## Functions

官方文档：[Functions and Operators](https://dev.mysql.com/doc/refman/8.0/en/functions.html)。

SQL 中的函数有很多类别，常用的有：

* [数值函数](https://dev.mysql.com/doc/refman/8.0/en/numeric-functions.html)：
  * ABS
  * MOD
  * ROUND
* [字符串函数](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html)：
  * CONCAT
  * LENGTH
  * CHAR\_LENGTH
  * LOWER
  * UPPER
  * REPLACE
  * SUBSTRING
* [日期和时间函数](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html)：
  * CURRENT\_DATE
  * CURRENT\_TIME
  * CURRENT\_TIMESTAMP
  * EXTRACT
  * DATE
  * YEAR
  * MONTH
  * DAY
  * HOUR
  * MINUTE
  * SECOND
* [转换函数](https://dev.mysql.com/doc/refman/8.0/en/cast-functions.html)：
  * CAST
  * COALESCE
* [聚集函数](https://dev.mysql.com/doc/refman/8.0/en/group-by-functions-and-modifiers.html)：`FUN(*)`会统计所有数据行数，`FUN(col)`会忽略 col 为 NULL 值的行。
  * COUNT：COUNT\(DISTINCT col\) 会统计不同值得个数，同样也会忽略 NULL 值。
  * MAX
  * MIN
  * SUM
  * AVG

使用函数可能带来的问题：

* 不同 DBMS 支持的函数不一样，移植容易产生兼容性问题；
* 容易使用不当，导致查询不使用索引；

## Utility Statements

```sql
# 获取表信息
DESCRIBE {tb_name};
DESCRIBE app;
```

