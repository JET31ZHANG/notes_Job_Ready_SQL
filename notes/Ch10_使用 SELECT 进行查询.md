# 使用 SELECT 进行查询

目标：

- 用 SELECT 语句读取数据；
- 在 WHERE 子句中对字符串、数字和日期值进行过滤；
- 使用 LIKE 运算符根据模式进行筛选；
- 描述空值和使用其策略。

## 设置数据库

本课使用一个消费者投诉数据集。

1. 从 [www.wiley.com/go/jobreadysql](http://www.wiley.com/go/jobreadysql) 下载 consumer-complaints-schema-and-data.sql；
2. 使用 MySQL Workbench 或 MySQL 命令行在 MySQL 中执行该脚本。

## 使用 SELECT 关键字

### 对单个表使用 SELECT

```sql
-- 显示列
USE ConsumerComplaints;
DESCRIBE Complaint;

-- 10-2 列出 ConsumerCompaints 中的某些列的数据
USE ConsumerComplaints;

SELECT DateReceived, Product, Company, State
FROM Complaint;
```

### 使用 SELECT*

```sql
-- 10-3 列出所有列
USE ConsumerComplaints;

SELECT *
FROM Complaint;
```

注意：小心使用 * 进行查询，在实际应用中，当不需要全部数据时，选择所有内容是浪费的。

## 使用 WHERE 子句

常用 WHERE 运算符

| 表达式 |                        用途                        |                 示例                 |
| :-----: | :------------------------------------------------: | :-----------------------------------: |
|    =    |                        等号                        |             State = 'LA'             |
| !=、< > |                        不等                        |     State != 'LA' State < > 'LA'     |
|   AND   |                         和                         | State = 'LA' AND Product = 'Mortgage' |
|   OR   |                         或                         | State = 'LA' OR Product = 'Mortgage' |
|   IN   | 匹配一个值列表；这是 OR 条件列表的一种更简短的写法 |       State IN('LA','AZ','TX')       |
| NOT IN |                   不在值的列表中                   |     State NOT IN('LA','AZ','TX')     |

```sql
-- 10-4 使用 WHERE 子句
USE ConsumerComplaints;
-- 两个连字符是 SQL 注释，这行代码会被忽略
-- 如果你的查询有很多列，则可能需要通过换行来提高可读性
-- 空格将被忽略
SELECT
DateReceived,
Product,
Issue,
Company
FROM Complaint
WHERE State = 'LA';
```

```sql
-- 10-5 从 ConsumerComplaints 数据库中获取记录的查询
USE ConsumerComplaints;
SELECT *
FROM Complaint
WHERE State = 'LA'
AND (Product = 'Mortgage' OR Product = 'Debt collection');
```

```sql
-- 10-6 删除括号
USE ConsumerComplaints;
SELECT *
FROM Complaint
WHERE State = 'LA'
AND Product = 'Mortgage' OR Product = 'Debt collection';
```

### 过滤数值

常用的数值比较运算符

| 表达式 |          用途          |                       示例                       |
| :-----: | :--------------------: | :-----------------------------------------------: |
|    =    |          相等          |              CompaintId = 1,653,822              |
| !=、< > |          不等          | CompaintId != 1,653,822、CompaintId < > 1,653,822 |
|    >    |          大于          |                CompaintId > 10,000                |
|   >=   |        大于等于        |               CompaintId >= 10,000               |
|    <    |          小于          |                CompaintId < 10,000                |
|   <=   |        小于等于        |               CompaintId <= 10,000               |
| BETWEEN | 列值处于一定范围的当中 |        ComlaintId BETWEEN 1,000 AND 30,000        |

```sql
-- 10-7 使用数学比较
USE ConsumerComplaints;

SELECT 
    Product,
    Issue,
    Company,
    ResponseToConsumer
FROM Complaint
WHERE ConsumerDisputed = 1
AND ConsumerConsent = 1
AND Product NOT IN ('Mortgage', 'Debt collection');
```

### 过滤日期

日期比较运算符

| 表达式 |       用途       |                              示例                              |
| :-----: | :--------------: | :------------------------------------------------------------: |
|    =    |       相等       |                  DateReceived = '2017-07-04'                  |
| !=、< > |       不等       | DateReceived != '2017-07-04'<br />DateReceived <> '2017-07-04' |
|    >    |       大于       |                  DateReceived > '2017-07-04'                  |
|   >=   |     大于等于     |                  DateReceived >= '2017-07-04'                  |
|    <    |       小于       |                  DateReceived < '2017-07-04'                  |
|   <=   |     小于等于     |                  DateReceived <= '2017-07-04'                  |
| BETWEEN | 列值在一定范围中 |       DateReceived BETWEEN '2017-01-01' AND '2018-01-01'       |

### 模式匹配文本

特殊字符：
- %：匹配任意数量的字符，包括空字符；
- _：匹配任意单个字符。

**注意：如果要匹配字面值 “%” 或 “_”，必须在字符前加上反斜杠（转义字符）。**

模式匹配示例

|     表达式     |                              说明                              |                不匹配                |                                                匹配                                                |
| :------------: | :------------------------------------------------------------: | :----------------------------------: | :------------------------------------------------------------------------------------------------: |
|   LIKE 'A%'   |      匹配以字母A开头的字符串（默认情况下，不区分大小写）      | Banana<br /> @#&?! <br /> cream corn |                                     Apple<br /> A <br /> atom                                     |
|   LIKE 'a%c'   | 匹配以a开头，以c结尾，并且在两者之间可以有任意数量字符的字符串 |  a brick<br /> atom <br /> bucolic  | abc<br /> AC <br /> A1's bric-a-brac <br /> All is quiet. Calm yourself. <br /> Don't be dogmatic. |
| LIKE '%space%' |               匹配任意位置包括值 space 的字符串               |            apostrophe ace            |                     outerspace<br /> spaceship <br /> tab, space, and newline                     |
|    LIKE '%'    |              匹配所有字符串。所以，它不是特别有用              |                                      |                                 a spaceship<br />Any value works!                                 |
|   LIKE '_at'   |            匹配任意单个字符开头，并以at结尾的字符串            |    brat<br />spaceship<br />phat    |                                       cat<br />bat<br />sat                                       |
|   LIKE '___'   |               匹配长度恰好为3个字符的任意字符串               |    1<br />spaceship<br />too long    |                                   abc<br />!!!<br />cat<br />too                                   |

### NULL：“十亿美元级别的错误”

特殊值 NULL 表示一个未设置的值或缺失的信息。NULL 在 WHERE 子句中的很多运算符是无效的。

```sql
-- 以下 WHERE 语句都无法识别 SubProduct 列值为 NULL 值的记录
-- 10-8 无效的 WHERE 语句
USE ConsumerComplaints;
-- 这个查询根本不返回任何记录。
SELECT *
FROM Complaint
WHERE SubProduct = NULL;

-- 这个查询也不返回任何记录
SELECT *
FROM Complaint
WHERE SubProduct != NULL;

-- 依旧是空的
SELECT *
FROM Complaint
WHERE ComplaintId BETWEEN 15000 AND NULL;

-- 结果中没有 NULL 值
SELECT *
FROM Complaint
WHERE SubProduct IN ('Other mortgage', NULL);
```

**要找到 NULL 值，需要使用特殊运算符 IS。**

```sql
-- 代码清单 10-9 使用 IS NULL 或 IS NOT NULL 的有效 WHERE 语句
USE ConsumerComplaints;

-- 返回 278 行
SELECT *
FROM Complaint
WHERE SubProduct IS NULL;

-- 返回 722 行
SELECT *
FROM Complaint
WHERE SubProduct IS NOT NULL;

-- 返回 991 行
SELECT *
FROM Complaint
WHERE ComplaintId > 15000 OR CompaintId IS NULL;

-- 返回 391 行
SELECT *
FROM Complaint
WHERE SubProduct = 'Other mortgage'
OR SubProduct IS NULL;

-- 所有的投诉的 ComplaintNarrative 列都应该有值
-- 排除那些空值
SELECT *
FROM Complaint
WHERE ComplaintNarrative IS NOT NULL;
```

[Null References: The Billion Dollar Mistake](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/)

## 执行计算

SELECT 查询可用于对现有数据（数字和日期）进行计算并生成新数据。


```sql
/*
 *  当前数据集有两个 DATE 列：
 *  - 收到投诉的日期(DateReceived)
 *  - 将投诉发送给公司的日期(DateSentToCompany)
 * */
-- 10-10 计算两个日期之间的天数
USE ConsumerComplaints;

SELECT 
    ComplaintId,
    DateReceived,
    DateSentToCompany,
    (DateSentToCompany -DateReceived) AS DateDifference
FROM Complaint;
```

```sql
-- 10-11 在 WHERE 子句中使用计算值：显示日期差超过 365 天的结果
USE ConsumerComplaints;

SELECT
    ComplaintId,
    DateReceived,
    DateSentToCompany,
    (DateSentToCompany -DateReceived) AS DateDifference
FROM Complaint
WHERE (DateSentToCompany -DateReceived) > 365;
```

```sql
-- 对 10-11 的一种替代
-- 10-12 在 WHERE 子句中使用计算表达式
/*
 *  对主 SELECT 语句的 FROM 语句进行调整，来使用从 Compalint 表中 SELECT 的记录，以及用于确定日期差异的计算。
 *  FROM 子句创建一个名为 Newtable 的新表（临时表），该表具有一个名为 DateDifference 的新列。
 *  在第一个 SELECT 中使用这个新表，检查计算列 DateDifference 是否大于 365。
 * */
USE ConsumerComplaints;

SELECT
    Newtable.ComplaintId,
    Newtable.DateReceived,
    Newtable.DateSentToCompany,
    Newtable.DateDifference
FROM (SELECT
        ComplaintId,
        DateReceived,
        DateSentToCompany,
        (DateSentToCompany -DateReceived) AS DateDifference
FROM Complaint) AS Newtable
WHERE Newtable.DateDifference > 365;
```

## 小结

- SELECT 语句从数据库读取数据，从一个或多个表中获取数据，对结果进行过滤；
- 可列出要包括的列，或使用(*)选择所有的列，列名之后跟着FROM子句，该子句包括要从哪些表中读取数据；
- WHERE 子句紧随在 FROM 子句后，是个布尔表达式。可使用 AND、OR 和其他运算符来组合多个运算符；
- 可使用表中现有数据进行计算，创建新的数据。AS 关键字可用于命名包括计算数据的列。

## 本课练习

### 练习 10-1：投诉
在本课中，有许多建议的问题，可以使用 SELECT 语句和 Consumer Complaints 数据库来回答。如果在阅读本课时没有这样做，则现在应编写适当的 SELECT 语句。
- ComplaintId 为 1,200,385 的记录是否存在？
- ComplaintId 小于 100,000 的投诉有多少？
- ComplaintId 在 100,000 和 200,000 之间，投诉最多的产品是什么？
- 2014 年元旦有没有人投诉？
- 2018 年有人投诉吗？
- 2015 年 7 月多少投诉？
- 有没有投诉的接收日期（DateReceived）早于发送日期（DateSentToCompany）?
- 查找以 V 开头的公司名称的消费者投诉。
- 查找在投诉中使用“whom”一词的投诉内容。
- 查找 SubmissionMethods 的长度恰好为三个字符的记录。
- 查找投诉中提到贷款（loan）的问题。

### 练习 10-2：私人教练
在这些练习中，你将使用 PersonalTrainer schema 完成一系列 SELECT 查询。这将建立 Personal Trainer 数据库。
你需要运行 personaltrainer-schema-and-data.sql 脚本来创建此数据库。可以在本书的配套文件中找到该脚本，可以在 www.wiley.com/go/JobReadySQL 上获取，或在 the-software-guild.s3.amazonaws.com/sql/v1-2003/data-files/personaltrainer-schema-and-data.sql 上找到它。
在将文件保存到计算机后，可以使用以下任何方法来运行此脚本：
- 打开MySQL Workbench，并连接到本地的 MySQL 服务器。双击保存的 .sql 文件，它应该会自动在 MySQL Workbench 中打开。使用工具栏中的 Execute 按钮来运行该脚本并创建数据库

- 在 MySQL Workbench 中，使用文件菜单或工具栏中的 Open SQL Script 命令。导航到保存在计算机上的文件，打开该文件，并单击 Execute 按钮运行该脚本。

- 打开 .sql 文件，并将其内容复制到系统剪贴板中。在 MySQL Workbench 中打开一个新的查询窗口，或者通过命令行界面（如 Windows 命令提示符或终端）连接到 MySQL 服务器。将脚本粘贴到提示符处，并运行它。

