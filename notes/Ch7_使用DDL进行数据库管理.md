# 使用 DDL 进行数据库管理

目标
- 创建和使用数据库；
- 生成一个关系数据库管理系统(DBMS)中可用数据库的列表；
- 从RDBMS中删除一个现有的数据库；
- 描述可用的数据类型，并将它们应用于表中的列；
- 在选定的数据库中生成表的列表；
- 显示现有表的结构；
- 更新表的结构；
- 从数据库中删除一个表；
- 生成一个可用于重建/复制数据库结构的脚本；
- 使用外键来定义表之间的关系；
- 理解什么是参照完整性；
- 识别和比较用于删除或更新主键列值的选项。

## 数据库管理

```sql
-- 创建新的数据库
CREATE DATABASE books;

-- 列出已经存在的数据库
SHOW DATABASES;

-- 使用数据库
USE books;

-- 删除一个现有的数据库
DROP DATABASE books;

DROP DATABASE IF EXISTS database_name;
```

## MySQL 数据类型

### 数据类型

常见的数据类型
- 整型
- 小数
- 字符串
- 日期/时间

### 数字数据类型

- 整数类型
|数据类型|有符号范围|无符号范围|存储空间|
|:-:|:-:|:-:|:-:|
|TINYINT|-128 ~ 127 |0 ~ 255|1字节|
|SMALLINT|-32,768 ~ 32,767|0 ~ 65,535|2字节|
|MEDIUMINT|-8,388,608 ~ 8,388,607|0 ~ 16,777,215|3字节|
|INT|-2,147,483,648 ~ 2,147,483,647|0 ~ 4294967295|4字节|

- 小数类型
|数据类型|存储空间|
|:-:|:-:|
|FLOAT|4字节|
|DOUBLE|8字节|
|DECIMAL|可修改精度|

DECIMAL 默认值是 DECIMAL(10,0) （总共10位有效数字，其中小数点右侧有0位数字，即10位整数）

### 字符串类型

字符串用于存储文本值。

|数据类型|描述|
|:-:|:-:|
|CHAR(size)|一个最多可以容纳 255 个字符的定长列|
|VARCHAR(size)|可变长度列，最多可存储 65,535 个字符|
|TINYTEXT|最大长度为 255 个字符的列|
|MEDIUMTEXT|最大长度为 16,777,215 个字符的列|
|LONGTEXT|最大长度为 4,294,967,295 个字符的列|

CHAR 是一个定长列，通常用于表示长度可能相同的字符串。CHAR 列总是为每条记录保留相同大小的空间，无论实际输入的值是什么。
VARCHAR 更适合存储不同长度的数据。VARCHAR 会根据当前值来调整所需的存储空间。

### 日期时间类型

|数据类型|描述|
|:-:|:-:|
|DATE|日期。默认格式为：YYYY-MM-DD|
|DATETIME|日期和时间的组合。默认格式为：YYYY-MM-DD hh:mm:ss|
|TIMESTAMP|时间戳。默认格式为：YYYY-MM-DD hh:mm:ss|
|TIME|时间。默认格式为：hh:mm:ss|
|YEAR|四位数格式的年份|

## 管理 MySQL 中的表

- 在数据库中创建表，包括定义表中的列和键；
- 在选定的数据库中生成表的列表；
- 查看现有表的结构；
- 更改表的结构；
- 从数据库中删除表。

### 创建表

```sql
-- 7-1 创建 book 表
USE books;
CREATE TABLE book  (
    bookId INT,
    bookTitle VARCHAR(100),
    numPage SMALLINT,
    origPubDate YEAR
);

/*
 *  bookId：
 *          主键值是一个随机分配的整数。INT 的取值范围很广泛，可用来存储大量书籍的ID值；
 *          默认情况下，任何主键列都是不能为空的。没必要额外说明它是不能为空的。
 *  bookTitle：
 *          每本书的标题长度不一样，应使用 VARCHAR 而不是 CHAR；
 *          基于预期的书名，使用 100 个字符是合理的最大值；
 *          数据库中每本书都应有一个书名，因此这个列不能为空。
 *  numPage：
 *          选择了 SMALLINT ，无符号值的最大值为 65,535。集合中的任何一本书都不应该超过这个页数；
 *          该列不是必需的，藏书可能有未知的书，该值可为空。
 *  origPubDate：
 *          用于存储书籍最早出版的年份；
 *          只需要 YEAR 值。
 * */
```

### 展示现有表

```sql
SHOW TABLES;
```

### 查看表

```sql
DESCRIBE book;
```

### 更改表

```sql
-- 删除列: numPages
ALTER TABLE book
DROP numPages;

-- 为现有表设定键
/* 
 *  主键名以 PK 开头，并包括作为主键的表的名称，
 *  在这种情况下，该键将被命名为 PK_book，
 *  还需要确定是哪种类型的约束，以及适用于哪些列。
 * */
ALTER TABLE book
ADD CONSTRAINT PK_book PRIMARY KEY (bookId);

-- 修改现有列
/*
 *  bookTitle 不能为空，使用 ALTER TABLE 命令和 MODIFY 子句进行更改。
 *  当一个列被更改时，必须更改该列的所有方面，即使这些方面没有改变。
 * */
ALTER TABLE book
MODIFY bookTitle VARCHAR(100) NOT NULL;

-- 添加列
/*
 *  要添加一个列，使用 ALTER TABLE 命令和 ADD 子句进行添加。
 * */
ALTER TALBE book
ADD genre VARCHAR(20);

-- 对已有数据的表进行更改
/*
 *  如果表中包括数据，则可能无法更改列。
 *  如果要声明一个 NOT NULL 列，且数据包含空值，MODIFY 操作将失败。
 *  如果将列更改为使用更严格的数据类型（如，将数据类型由 VARCHAR(50) 修改为 VARCHAR(10)），
 *  并且列包括长度超过 10 个字符的值，则 MODIFY 操作也将失败。
 *  如果要强制更改，则须更新数据来符合限制，再进行 MODIFY 操作。
 * */
```

### 删除表

```sql
-- 删除表 book
/* 删除一个表，不仅会删除该表本身，还会删除它可能包括的所有数据 */
DROP TABLE book;

/* 使用 SHOW 命令验证该表是否已从数据库中删除 */
SHOW TABLES;
```

### 总结book表的变化

一旦数据保存到表中，再更改表的时候可能会对表中的数据进行更改或删除。

```sql
-- 7-2 创建 book 表
/* 最初应该使用的适当的 CREATE TABLE 语句 */
CREATE TABLE book (
    bookId INT NOT NULL,
    bookTitle VARCHAR(100) NOT NULL,
    origPubDate YEAR,
    CONSTRAINT PK_book PRIMARY KEY (bookId)
);
```

## 管理 MySQL 中的关系

### 定义外键

两个表之间的关系：通过将一个表的主键用作另一个表的外键来定义的。

```sql
-- 7-3 创建 person 表
CREATE TABLE person (
    person_id INT,
    first_name VARCHAR(25),
    last_name VARCHAR(50) NOT NULL,
    birthday DATE,
    CONSTRAINT PK_person PRIMARY KEY (person_id)
);
```

```sql
-- 7-4 创建 phone 表
CREATE TABLE phone (
    phone_id INT,
    phone_number VARCHAR(20) NOT NULL,
    phone_type VARCHAR(15) NOT NULL,
    person_id INT NOT NULL,
    CONSTRAINT PK_phone PRIMARY KEY (phone_id),
    CONSTRAINT FOREIGN KEY FK_person_phone (person_id)
        REFERENCES person (person_id)
);

/*
 *  外键：通常在名称前加上 FK，并包括两个相关表的名称。
 *  外键约束还要求 REFERENCES 子句标识出它所表示的是哪个表中的哪个列。
 *
 *  大多数情况下，相关列在两个表中具有相同的名称，但有时外键需要不同的列名称。
 *  比如，在同一个表中有主键和外键列的自引用表。
 * */
```

### 实体完整性

定义主键后，RDBMS 会自动对新记录应用实体完整性约束，包括：
- 表中没有其他现有记录与新条目具有相同的主键值；
- 主键对应的列不能有空值。

主键通常设置为整数值。
当列被添加到列表中时，通常会设置为自动编号的记录。这样只能保证“记录”不重复，但不能保证“数据”不重复。

### 参照完整性

实体完整性只适用于一个表中的记录，参照完整性适用于添加到相关表中的记录。
**任何用作外键的值都必须在相关表中作为主键存在。**

参照完整性状态：
- 向外键添加数据：当添加外键约束时，RDBMS 会检查每个输入到外键列中的值是否已经存在于另一个表的相关主键中。即当向数据库添加数据时，必须先将数据添加到主键所在的表，然后才能将其添加到相关表中。
- 更新主键记录的数据：不会经常更新主键值。更改这些值将违反关系的完整性，因为外键值不再与主键列中的值匹配。
- 删除主键记录

### 参照完整性的解决方案

删除主表中的多条记录，可使用的选项：
- 删除外键约束：简单地临时删除外键约束，并在对数据进行必要的更改后重新应用它们。但是，重新应用约束时，可能某些外键值不再与现有的主键值匹配，并需要查找和纠正错误。**在向数据库添加数据时，最好设置一个外键约束，确保所有的外键值都满足参照完整性的要求。**

- 使用 ON UPDATE：在更改主键值时，在外键约束中加入 ON UPDATE CASCADE 选项。设置此选项后，当改变已有数据中的主键值时，RDBMS 会自动更新相关外键值，以便在使用给定主键值的任何地方自动发生更改。大多数情况下，应选择主键列，使其值不太可能改变。**选择主键时，一个无意义的替代键比用户名更好**（用户更改用户名，可以允许更改而不更改主键。）

- 使用 ON DELETE：ON DELETE CASCADE 选项有破坏性（从主表删除一条记录，则基于该主键的所有相关记录也将从相关表中删除）。可改为 ON DELETE SET NULL 只删除相关记录的外键值，此选项有效删除了表之间的关系。如果需要保留表之间的关系，且外键设置为 NOT NULL，则不允许使用此选项。

在向数据库添加数据时，应遵循：
- 向数据库添加数据前，确保每个表定义了关系。（在相关表中定义外键）
- 向数据库添加新数据时，将数据添加到相关表之前先将其添加到主表中，避免违反参照完整性。

## 小结

- SQL 语言有两种类型：
    - 数据定义语言DDL：定义和创建数据库及其结构（表、索引、表之间的关系）；
    - 数据操作语言DML：操作存储在存储器中的数据。

- 通过 DDL：
    - 学习了创建数据库的核心方法（添加表和列）；
    - 如何为这些列分配数据类型；
    - 如何设定他们可以为空或不可为空；
    - 通过定义与使用主键和外键来管理数据库中表之间的关系。