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

## DDL

```sql
# 获取表信息
decribe {tb_name};
decribe app;

# 增加列
ALTER TABLE {tb_name}
ADD [COLUMN] {col_name1} {col_definition1} [FIRST|AFTER existing_column1],
ADD [COLUMN] {col_name2} {col_definition2} [FIRST|AFTER existing_column2],...;

ALTER TABLE app ADD created_by varchar(32) NULL COMMENT '创建人';

# 删除列
ALTER TABLE {tb_name}
DROP COLUMN {col_name1},
DROP COLUMN {col_name2},...;

ALTER TABLE app DROP COLUMN created_by;

DROP schema XXX;
```

