# SQL 快速参考

SQL 基本语法
- 使用数据库
- 定义表、列和行
- 在表上执行查询
    - 对结果进行过滤和分组
    - 多表查询
- 基本的 SQL 数据类型

## 使用数据库

- 创建新数据库

```sql
CREATE DATABASE databaseName;
```

- 使用现有数据库

```sql
USE databaseName;
```

- 删除数据库

```sql
DROP DATABASE databaseName;
```

## 定义表、列和行

- 创建新表

```sql
CREATE TABLE tableName
(
    columnName1 dataType,
    columnName2 dataType,
...
);
```

- 删除表

```sql
-- 方法1
DROP TABLE tableName;

-- 方法2
DELETE FROM tableName
WHERE columnName = value;

-- 方法3
DELETE * FROM tableName;

-- 方法4
DELETE FROM tableName;
```

- 重命名表

```sql
-- 方法1
ALTER TABLE originalTableName RENAME TO newTableName;

-- 方法2
RENAME TABLE originalTableName TO newTableName;
```

- 向表中添加一个新列

```sql
ALTER TALBE tableName
ADD [COLUMN] columnName datatype;
```

- 从表中删除一个列

```sql
ALTER TABLE tableName
DROP [COLUMN] columnName;
```

- 向表中添加一个列

```sql
ALTER TABLE tableName
ADD newColumnName datatype [columnConstraint(s)] [AFTER existingColumn];
```

- 更改表中的列

```sql
ALTER TABLE tableName
MODIFY columnName [columnDefinition] [columnConstraint(s)];
```

- 将数据行添加到表中

```sql
INSERT INTO tableName [(columnName1, columnName2, ... ;)]
VALUES (value1, value2, ...);
```

- 将多个数据行插入到表中

```sql
INSERT INTO tableName [(columnName1, columnName2, ... ;)]
VALUES (row1Value1, row1Value2, ...),
       (row2Value1, row2Value2, ...),
       ... ;
```

- 在表中更新一个值

```sql
UPDATE tableName
SET columnName = value [, columnName2 = value2]
WHERE condition;
```

- 创建视图

```sql
CREATE VIEW viewName AS
SELECT columnName(s)
FROM tableName
WHERE condition;
```

- 创建索引

```sql
CREATE [UNIQUE] INDEX indexName
ON tableName (columnName);
```

- 删除索引（MySQL）

```sql
DROP INDEX indexName;
```

## 在表上执行查询

- 从表中选择所有列

```sql
SELECT *
FROM tableName;
```

- 从表中选择特定列

```sql
SELECT columnName1, columnName2, columnNameX
FROM tableName;
```

- 按升序（ASC）或降序（DESC）顺序排序

```sql
SELECT *
FROM tableName
ORDER BY column1 [ASC | DESC];
```

### 对结果进行过滤和分组

- 使用比较运算符进行过滤

```sql
SELECT *
FROM tableName
WHERE BooleanCondition [AND BooleanCondition2] [OR 
BooleanCondition3];
```

- 使用 LIKE 运算符进行过滤

```sql
SELECT *
FROM tableName
WHERE columnName LIKE pattern;
```

- 基于 ID 进行过滤

```sql
-- 方法1
SELECT *
FROM tableName
WHERE keyField_Id IS value;

-- 方法2
SELECT *
FROM tableName
WHERE keyField_Id IN (value1, value2, ...);
```

- 基于是否为空进行过滤

```sql
SELECT *
FROM tableName
WHERE columnName IS NOT NULL;
```

- 基于范围进行过滤

```sql
SELECT *
FROM tableName
WHERE columnName BETWEEN value1 AND value2;
```

- 使用聚合函数进行过滤

```sql
SELECT *
FROM tableName
WHERE columnName BETWEEN value1 AND value2;
```

- 基本分组：

```sql
SELECT *
FROM tableName
[WHERE columnName operator value]
GROUP BY columnName [, columnName2]; 
```

### 多表查询

- 使用 INNER JOIN(基本的 JOIN)

```sql
SELECT *
FROM tableName1
[INNER] JOIN tableName2
    ON tableName1.columnName = tableName2.columnName;
```

- 使用 LEFT JOIN

```sql
SELECT *
FROM tableName1
LEFT JOIN tableName2
    ON tableName1.columnName = tableName2.columnName;
```

- 使用 RIGHT JOIN

```sql
SELECT *
FROM tableName1
RIGHT JOIN tableName2
ON tableName1.columnName = tableName2.columnName;
```

- 使用 FULL JOIN

```sql
SELECT *
FROM tableName1
FULL JOIN tableName2
ON tableName1.columnName = tableName2.columnName;
```

- 使用 CROSS JOIN

```sql
SELECT *
FROM tableName1
CROSS JOIN tableName2;
```

## 基本的 SQL 数据类型

- CHARACTER 或 CHAR：保存单个字符
- CHARACTER(n) 或 CHAR(n)：最多可保存 n 个字符
- VARCHAR(n) 或 CHARACTER VARYING(n)：最多可保存 n 个字符
- BIT(n) 或 BIT VARYING(n)：最多可保存 n 位。一位可以是0或1
- DECIMAL(p,s)：保存一个数值，其中，p 是精度（位数），s 是标度（小数点后的位数）
- INT 或 INTEGER：保存一个整数值
- SMALLINT：保存一个较小的整数值
- BIGINT：保存一个较大的整数值
- FLOAT(p,s)：保存一个浮点值，其中，p 是精度（位数），s 是标度（小数点后的位数）
- REAL(s)：保存一个近似浮点数。REAL 与 FLOAT(24) 相同
- DATE：保存一个标识日期的值，通常为年、月、日值，范围一般为 0001-01-01 ~ 9999-12-31
- TIME：保存一个表示一天中时间的值，格式为小时、分钟和秒，并可以存储可选的纳秒值。使用 “HH:MM:SS.nnnn” 的格式
- TIMESTAMP：保存一个组合的日期和时间值，格式为 “YYYY-MM-DD HH:MM:SS”