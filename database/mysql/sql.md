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

### VIEW

```sql
# 创建视图
CREATE VIEW view_name AS select_statement;

CREATE VIEW player_above_avg_height AS
  SELECT player_id, height
  FROM player
  WHERE height > (SELECT AVG(height) from player);
  
# 视图可以嵌套
CREATE VIEW player_above_above_avg_height AS
  SELECT player_id, height
  FROM player
  WHERE height > (SELECT AVG(height) from player_above_avg_height);
  
# 修改视图
ALTER VIEW view_name AS select_statement;

ALTER VIEW player_above_avg_height AS
  SELECT player_id, player_name, height
  FROM player
  WHERE height > (SELECT AVG(height) from player);
  
# 删除视图
DROP VIEW view_name;
```

视图的特点：

* 可用于安全性，比如我们需要对表的字段级别做权限设置，那么可以用视图来实现。
* 简化 SQL，编写好复杂查询的视图后，我们仅需要简单的查询就行。
* 是虚拟表，本身不存储数据，所以对数据的修改限制很多，所以视图一般用作查询。

### PROCEDURE

存储过程和视图一样，也是对 SQL 代码的封装，可以反复利用。不同的是，视图是虚拟表，通常不用来操作数据，而存储过程是程序化的 SQL，可以操作底层数据。存储过程由 SQL 语句和流程控制语句构成。

```sql
CREATE PROCEDURE sp_name ([proc_parameter[,...]])
BEGIN
    需要执行的语句
END

DROP {PROCEDURE | FUNCTION} [IF EXISTS] sp_name
```

比如计算 1 到 n 的和：

```sql
DELIMITER //

CREATE PROCEDURE `add_num`(IN n INT)
  BEGIN
    DECLARE i INT;
    DECLARE sum INT;
    SET i = 1;
    SET sum = 0;
    WHILE i <= n DO
      SET sum = sum + i;
      SET i = i + 1;
    END WHILE;
    SELECT sum;
  END //

DELIMITER ;

CALL add_num(50);
```

{% hint style="info" %}
默认 SQL 采用 ; 作为结束符，这样存储过程中的每一句 SQL 都需要执行。所以通过 DELIMITER 临时修改结束符，存储过程创建好后再变回来。若是 Navicat 等工具就不需要，它会自动帮你设置 DELIMITER，用 MySQL client 就需要。
{% endhint %}

#### 参数类型

* IN：向存储过程传入参数，不返回。
* OUT：把存储过程计算结果放入该参数，调用者可以得到返回值。
* INOUT：IN 和 OUT 的结合。

```sql
CREATE PROCEDURE `get_hero_scores`(
  OUT max_max_hp FLOAT, 
  OUT min_max_mp FLOAT, 
  OUT avg_max_attack FLOAT, 
  s VARCHAR(255))
  BEGIN
    SELECT MAX(hp_max), MIN(mp_max), AVG(attack_max)
    FROM heros
    WHERE role_main = s INTO max_max_hp, min_max_mp, avg_max_attack;
  END
  
CALL get_hero_scores(@max_max_hp, @min_max_mp, @avg_max_attack, '战⼠');
SELECT @max_max_hp, @min_max_mp, @avg_max_attack;
```

#### 流程控制语句

* **BEGIN ... END**：中间包含多个语句，每个语句以 ; 结束。
* **DECLARE**：申明变量。
* **SET**：赋值。
* **SELECT ... INTO**：赋值。
* **IF ... THEN ... \[ELSE\|ELSEIF\] ... ENDIF**：条件判断。
* **CASE**：多条件分支判断。
* **LOOP、LEAVE、ITERATE**：LEAVE 可理解为 break，ITERATE 可理解为 continue。
* **REPEAT ... UNTIL ... END REPEAT**：可理解为 do while。
* **WHILE ... DO ... END WHILE**：可理解为 while。

#### 优缺点

* 优点：
  * 一次编译多次使用
  * 代码封装成模块，复杂问题可以拆解成简单问题
  * 模块之间可重复使用
  * 安全性强，可以为存储过程设定权限
  * 可减少网络传输，包括代码数据量、连接次数等
* 缺点：
  * 移植性差，很难跨数据库移植
  * 调试困难
  * 数据表索引发生变化，可能导致存储过程失效
  * 不适合高并发，因为高并发数据库会分库分表

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

通配符：`LIKE`，查询英雄名字除了第一个字以外，包含"太"字的英雄：

```text
SELECT name FROM heros WHERE name LIKE '_%太%';
```

\_ 匹配任意一个字符，% 匹配大于等于 0 个字符。所以“太乙真人”匹配不上，“东皇太乙”可以匹配，“太乙真人太太”可以匹配。

另外 `LIKE '%'` 无法查出 NULL 值。

### LIMIT

LIMIT 最后执行。

如果确定查询结果只有一条，那么需不需要加 LIMIT 1？有两种情况

* 若对某个字段建立了唯一索引，那么对这个字段查询，不需要加 LIMIT 1.
* 若是全表扫描的 SQL 语句，若加了 LIMIT 1，当找到一条结果后就不会继续扫描了，能加快查询速度。

### ORDER BY

尽量在 ORDER BY 字段上加索引，为什么？

MySQL 有两种排序方式，FileSort 和 IndexSort：

* FileSort：一般在内存中排序，CPU 占用较多，如果待排序结果较大，还会产生临时文件到磁盘，效率较低。
* IndexSort：索引可以保证数据的有序性，不需要进行排序。

所以当使用 ORDER BY 时，应尽量使用 IndexSort，可以使用 explain 查看是否使用索引排序。

所以有如下优化建议：

* WHERE 和 ORDER BY 都使用索引，WHERE 使用时为了避免全表扫描，ORDER BY 使用时为了避免 FileSort。
* 若 WHERE 和 ORDER BY 是相同的列，就使用单索引列；若不同，就使用联合索引。

### GROUP BY

我们可以对数据进行分组，并使用聚集函数统计每组数据的值。

```sql
SELECT COUNT(*) AS num, role_main, role_assist FROM heros GROUP BY role_main, role_assist ORDER BY num DESC;
```

若 GROUP BY 后面还有 ORDER BY，那么排序实际上是对分组后的数据排序，因为分组已经把多条数据聚合成了一条记录。

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
# 先执行子句，在执行外层 SQL
for i in B
    for j in A
        if j.cc == i.cc then ...

SELECT * FROM A WHERE EXISTS (SELECT cc FROM B WHERE B.cc=A.cc)
# 对 A 表遍历，每条记录执行子句进行判断
for i in A
    for j in B
        if j.cc == i.cc then ...
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

#### USING

用于指定数据表中的同名字段进行等值连接。可以简化 JOIN ON 的等值连接。

```sql
# SQL99
SELECT player_id, team_id, player_name, height, team_name
FROM player
       JOIN team USING (team_id);
```

#### LEFT \[OUTER\] JOIN

左边是主表，需要显示左边表的全部行。

```sql
# SQL99
SELECT *
from player
       LEFT JOIN team USING (team_id);

# SQL 92, MySQL 不支持
SELECT *
FROM player,
     team
where player.team_id = team.team_id(+);
```

#### RIGHT \[OUTER\] JOIN

右边是主表，需要显示右边表的全部行。

```sql
# SQL99
SELECT *
FROM player
       RIGHT JOIN team USING (team_id);

# SQL 92, MySQL 不支持
SELECT *
FROM player,
     team
where player.team_id(+) = team.team_id;
```

#### FULL \[OUTER\] JOIN

MySQL不支持。全外连接的结果 = 左右表匹配的数据 + 左表没有匹配到的数据 + 右表没有匹配到的数据。

```sql
SELECT * FROM player FULL JOIN team ON player.team_id = team.team_id;
```

#### 自连接

自连接的速度比子查询快很多，所以建议尽量使用自连接。

```sql
SELECT b.player_name, b.height
FROM player as a
       JOIN player as b ON a.player_name = '布雷克-格⾥芬' and a.height < b.height;
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

### COUNT 效率

结论：**`COUNT(*) = COUNT(1) > COUNT(col)`**

* COUNT\(\*\) 和 COUNT\(1\) 都是针对对所有结果，若没有 WHERE 则是所有行，若有则是所有符合筛选条件的行。因此两者都是 O\(N\)，采用全表扫描，循环+计数。
* 但若是 MyISAM 引擎，则复杂度为 O\(1\)，因为 MyISAM 表有 meta 信息，存储了 row\_count，一致性由表级锁保证。InnoDB 支持事务，采用行级锁和 MCC 机制，所以没法维护 row\_count，需要扫描全表。
* 使用 InnoDB 时，若采用 COUNT\(\*\) 和 COUNT\(1\)，尽量在表上建立二级索引，因为主键是聚簇索引，包含的信息较多。这两个不需要查找具体行，只需要行数，系统会自动使用 key\_len 小的二级索引，若没有二级索引，才会使用主键索引。
* 若要使用具体的行，采用主键索引效率更高。

## Cursor

官方文档，[cursors](https://dev.mysql.com/doc/refman/8.0/en/cursors.html)。

```sql
DECLARE cursor_name CURSOR FOR select_statement;
OPEN cursor_name;
FETCH cursor_name INTO var_name ...;
CLOSE cursor_name;
DEALLOCATE PREPARE;
```

## Utility Statements

```sql
# 获取表信息
DESCRIBE {tb_name};
DESCRIBE app;
```

