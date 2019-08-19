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

## Utility Statements

```sql
# 获取表信息
DESCRIBE {tb_name};
DESCRIBE app;
```

