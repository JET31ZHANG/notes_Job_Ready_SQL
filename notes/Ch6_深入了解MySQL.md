# 深入了解 MySQL

目标：

- 讨论 SQL 的起源；
- 演示核心 SQL 的语法特性，包括字母大小写、空格、引号和分号的使用；
- 解释什么是数据集内的 null 值；
- 向表中添加 null 值；
- 解释关系数据库如何使用索引来提高检索效率；
- 确定可能影响索引的设计特性。

## SQL 简介

结构化查询语言 SQL
语句（statement）：完整的命令。
查询（query）：从数据库中检索数据的语句。

## SQL 语法

```sql
-- 基本语法：
SELECT field1, field2, field3
FROM table1
WHERE criteria
ORDER BY field1, field2;
```

### 分号

标准SQL语句总是以分号结尾

- MySQL 要求在每个语句的结尾使用一个分号或/g
- T-SQL 的最新版本不需要分号，但它们支持分号，即包含分号无妨，
  MySQL 中，每条SQL语句的结尾处都会使用分号

### 换行和缩进

在 SQL 语句中不需要换行或缩进，但使用它们可提高代码的可读性

```sql
-- 
create table `Client` (ClientId char(36) primary key, FirstName varchar(50)
not null, LastName varchar(50) not null, BirthDate date null, Address 
varchar(256) null, City varchar(100) null, StateAbbr char(2) null, PostalCode
varchar(10) null, foreign key fk_Client_StateAbbr (StateAbbr) 
references
State(StateAbbr));

CREATE TABLE  `Client` (
    ClientId CHAR(36) PRIMARY KEY, 
    FirstName VARCHAR(50) NOT NULL, 
    LastName VARCHAR(50) NOT NULL, 
    BirthDate DATE NULL, 
    Address VARCHAR(256) NULL, 
    City VARCHAR(100) NULL, 
    StateAbbr CHAR(2) NULL, 
    PostalCode VARCHAR(10) NULL, 
    FOREIGN KEY fk_Client_StateAbbr (StateAbbr) 
        REFERENCES State(StateAbbr)
);
```

```sql
-- SELECT 语句从指定表的指定列中检索数据
SELECT FirstName, LastName, Address FROM Client;

-- 将语句写在两行中，更容易阅读
SELECT FirstName, LastName, Address 
FROM Client;
```

### 字母大小写

SQL 关键字不区分大小写，将关键字写成大写字母可提高可读性。

### 逗号

SQL 使用逗号来分隔序列中的类似对象。

```sql
-- CREATE TABLE 语句中，每个列定义后面都有一个逗号
CREATE TABLE  `Client` (
    ClientId CHAR(36) PRIMARY KEY, 
    FirstName VARCHAR(50) NOT NULL, 
    LastName VARCHAR(50) NOT NULL, 
    BirthDate DATE NULL, 
    Address VARCHAR(256) NULL, 
    City VARCHAR(100) NULL, 
    StateAbbr CHAR(2) NULL, 
    PostalCode VARCHAR(10) NULL, 
    FOREIGN KEY fk_Client_StateAbbr (StateAbbr) 
        REFERENCES State(StateAbbr)
);

-- 序列的最后一项（Address）没有逗号
SELECT FirstName, LastName, Address
FROM Client;
```

### 空格

SQL 要求语句中每个逻辑词（包含关键字和对象名）后面都有一个空格。逗号或括号周围不需要空格，但在其周围使用空格可提高可读性。
任何RDBMS的表、列和索引等对象的名称不应包含空格。若对象的名称无论如何要包含空格，则名称必须放在引号中。

```sql
SELECT "First Name", "Last Name", Address
FROM Client;
```

### 引号

可以使用双引号或单引号，只要成对出现即可，但不能混用。

```sql
-- 查找姓氏为 Smith 的客户端
SELECT FirstName, LastName, Address
FROM Client
WHERE LastName = "smith";

SELECT FirstName, LastName, Address
FROM Client
WHERE LastName = 'smith';

-- 以下查询将无法工作
SELECT FirstName, LastName, Address
FROM Client
WHERE LastName = 'smith";
```

### 拼写问题

所有关键字必须拼写正确，所有对象名称的拼写必须与他们在数据库中出现的方式完全相同（即使它们在数据库中可能拼写错误）。

## 处理空值

null 值是一个空值，它不包括任何值。

### null 与 0

null 和 0 不一样。0是一个值，而null不是。

### 可以为空的列

可以为空的列是一个可以有值也可以没有值的列。
主键中使用的任何列都不能为空。实体完整性要求每个主键都有一个值。
设计数据库时，必须考虑数据库本身的目的，以及每个列对数据库各个用户的重要性。

```sql
-- 6-1 创建 Product 表
/* 创建了一个名为 Product 的新表，
 * 定义了三个列：  
 *              ProductID 是一个字符串，为主键；
 *              ProductName 是一个最多25个字符的变量字符串；
 *              Price 是一个浮点数。
 * */
CREATE TABLE Product (
    ProductID INT NOT NULL PRIMARY KEY,
    ProductName VARCHAR(25) NOT NULL,
    Price FLOAT NOT NULL
);
```

### 空值的后果

如果表中可为空的列相对较少，并且即使这些列为空，表中的数据一九可以使用列，那么这是一个合理的权衡，因为它允许数据在未来被快速添加到这些列中。
如果一个表有相当多的可空列，则值得创建一个单独的表，只包括这些可以为空的列，从而允许用户仅在必要时使用这些列创建记录。
必须考虑减少空列能带来多少效率。

## 使用索引

一个好的索引应具备的特征

- 很容易找到
- 只包括用户可能要查找的值
- 对索引值进行排序，使用户可轻松找到特定的值

### 主存储与辅助存储

### 索引列

查看每个列的使用情况，只索引用户最有可能需要的列。

#### 默认索引

默认情况，RDBMS 会自动索引关键列，包括主键和外键。这些列对于跨表检索数据至关重要，同时，对这些列建立索引有助于提高查询效率。
当从表中检索数据而不指定排序顺序时，无论记录添加到表中的顺序如何，结果都将默认按主键值排序。
不必查看索引中的每个值才能找到所需内容。只需跳过索引的列或页面，直到接近要查找的内容，然后只在距离要查找的术语的几条记录内才关注单个值。同理，数据库采用常见的查找算法快速查找特定值。
使用自动递增的键添加记录有助于提高写入效率，因为它可以保证每个新纪录都将添加到主表的末尾。

#### 唯一索引和非唯一索引

索引还可以控制列中的允许值。

- 主键索引：按定义验证该列中使用的每个值在表内都是唯一的（通过在应具有唯一性的列上设置唯一性索引来实现，不必将该列定义为主键）；
- 外键索引：不检查唯一性，但会自动强制执行引用完整性。

## 小结
- 创建数据库的表时，要审查将包括的列。因为表中的列默认可以包括空值，故设计审查应包括确定列是否应该标记为 NOT NULL，来确保需要一个值。主键列 NOT NULL 是必须的。如果列可能包括大量空元素，则可能需要重新设计数据库，减少所需的存储空间。
- 为常用列分配索引，可提高查找效率。
- 数据库被打开时，索引列（主键和外键）中的数据被自动加载到内存中。

## 本课练习

### 练习 6-1：对 SQL 代码进行格式化
重新格式化下面这行 SQL 代码，使其呈现出更清晰、更易于阅读的形式。

```sql
SELECT Last, First, Email, MobileNumber FROM Contacts WHERE Age >= 21 ORDERED
BY last, First;
```

```sql
SELECT Last, First, Email, MobileNumber 
FROM Contacts 
WHERE Age >= 21 
ORDER BY last, First;
```

### 练习 6-2：联系人问题
请看下面列表：

```sql
CREATE TABLE Contact (
    ID INT NOT NULL PRIMARY KEY,
    Last VARCHAR(50) NOT NULL,
    First VARCHAR(40),
    Age INT,
    Email VARCHAR(100),
    MobileNumber VARCHAR(12),
    HomeNumber VARCHAR(12),
    WorkNumber VARCHAR(12)
);
```

回答下列问题：
- 列表中哪些列是必须提供数据的？
- 每个列的数据类型是什么？
- 一个列能存储的最长姓氏是多少个字符？
- Age 列是必须给出值的吗？
- 要使 WorkNumber 成为必须提供值的列，需要更改什么？

### 练习 6-3：丢失联系人
在本科介绍了 null。如表 6-7 所示的联系人数据库允许使用空值。然而，这个数据库浪费了很多空间。重新设计此数据库，使其以一种更有效地利用存储空间的方式使用多个表。

表 6-7 联系人（Contacts） 表
|ID|Last|First|Email|MobileNumber|HomeNumber|WorkNumber|Fax|
|:-|:-|:-|:-|:-|:-|:-|:-:|
|c001|Jones|John|john@bogus.com|317-555-1212|317-555-1213|317-555-1214|null|
|c002|Buford|Bob|null|null|null|415-555-3333|null|
|c003|Smith|Sam|Sam@bogus.com|null|415-555-1212|null|null|
|c004|Michaels|Mitch|null|415-555-2121|null|null|null|
|c005|Andrews|Adam|Adam@bogus.com|698-5555-1212|null|null|null|
|c006|Finkelstein|Fred|null|null|217-555-4340|null|null|
|c007|Black|Brent|brent@bogus.com|null|null|null|null|