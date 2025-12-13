# 深入探讨高级 SQL 主题

目标：
- 使用简单的子查询；
- 解释视图的利弊；
- 解释数据库事务处理的需求；
- 使用优化技术来提高 MySQL 数据库的性能；
- 描述如何使用索引来提高数据库性能。

## 添加子查询

子查询：嵌入到另一个查询中，产生一个值或表结果集的查询。即使将其从父查询中分离，仍能独立运行。

子查询的使用范围：
- IN 子句
- 用子查询替代一个表或一个值

使用 trackit-schema-and-data.sql 

### IN 运算符与子查询

```sql
-- IN 运算符中的值来自查询
-- 15-1 获取被分配到某个项目的所有工人
/*
 *  WorkerId 根据其被提及的位置表示两个不同的含义：
 *  1. 在主查询中 Worker.WorkerId
 *  2. 在IN子句内部 ProjectWorker.WorkerId
 * */
USE TrackIt;

SELECT * 
FROM Worker
WHERE WorkerId IN (
    SELECT WorkerId FROM ProjectWorker
);

/*
 *  本例中，如果使用 JOIN 操作会返回重复的工人，因为一个工人可以被分配到多个项目。
 *  但 IN 方法不会产生重复的工人。
 *
 *  注意： 当子查询返回大量结果时，IN（子查询）的性能表现不佳。
 *        这种情况下，使用 JOIN 和 GROUP 将更快。
 *        如果子查询返回的值远远超过 100 个，则不要使用 IN（子查询）。
 * */
```

### 将子查询用作表

查询中的任意一个表都可以被子查询替代：
- 在子查询的基础上构建一个次级 SELECT 语句；
- 将子查询与一个表进行 JOIN 操作；
- 将一个子查询与另一个子查询进行 JOIN 操作。

```sql
-- 同时获取一个项目及添加到该项目的第一个任务的情况
-- 15-2 存在问题的查询语句
/* 
 *  存在问题的查询语句：没能获取第一个任务的列
 *
 *  唯一可以选择的值是分组的 Project 列和 Task 的聚合结果。
 *  没有一个聚合函数可以从一个特定的记录中获取一个列。
 * */
SELECT 
    p.Name ProjectName,
    MIN(t.TaskId) MinTaskId
    -- 我们需要的是 t.Title, 但是 SQL 引擎不知道我们指的是哪个任务（Task）
-- t.Title 不是分组的一部分，也没有聚合函数来确保获取 MinTaskId 对应的 Title
FROM Project p
INNER JOIN Task t ON p.ProjectId = t.ProjectId
GROUP BY p.ProjectId, p.Name;
```

```sql
-- 15-3 使用子查询解决问题
/*
 *  15-2 的查询变为子查询，与任务 Task 表进行连接，因为子查询没有名称，故赋予别名 g；
 *  子查询可以在 ON 条件、WHERE 条件和 SELECT 值列表中使用，并保留它们的别名。
 * */
SELECT 
    g.ProjectName,
    g.MinTaskId,
    t.Title MinTaskTitle
FROM Task t
INNER JOIN (
SELECT
    p.Name ProjectName,
    MIN(t.TaskId) MinTaskId
FROM Project p
INNER JOIN Task t ON p.ProjectId = t.ProjectId
GROUP BY p.ProjectId, p.Name) g ON t.TaskId = g.MinTaskId;
```

### 将子查询用作值

任意一个列或值都可以被子查询替代。

```sql
-- 15-4 查询工人信息，以及分配给它们的项目数量
/*
 *  子查询直接嵌入到 SELECT 值列表中。
 *
 *  WorkerId 是 ProjectWorker.WorkerId，而 w.WorkerId 是 Worker.WorkerId 的别名。
 * */
SELECT 
    w.FirstName,
    w.LastName,
    (SELECT COUNT(*) FROM ProjectWorker
    WHERE WorkerId = w.WorkerId) ProjectCount
FROM Worker w;
```

```sql
-- 15-5 通过子查询解决问题
/*
 *  将子查询当作值来使用，性能通常不好。
 *  如果查询在 SELECT 列表中被定义，则它会为结果中的每一条记录运行一次。
 * */
SELECT 
    p.Name ProjectName,
    MIN(t.TaskId) MinTaskId,
    (SELECT Title FROM Task
    WHERE TaskId = MIN(t.TaskId)) MinTaskTitle
FROM Project p
INNER JOIN Task t ON p.ProjectId = t.ProjectId
GROUP BY p.ProjectId, p.Name;
```

## 使用视图

视图：存储在数据库中一个命名的查询。它可以在 SELECT 语句任意一处被像表一样进行处理，也可以看作一个已命名的子查询。

视图遵循的DDL模式：CREATE objectType objectName。

视图的查询内容放在 AS 关键字后面。

```sql
-- 15-6 创建视图
CREATE VIEW ProjectNameWithMinTaskId
AS
SELECT 
    p.Name ProjectName,
    MIN(t.TaskId) MinTaskId
FROM Project p
INNER JOIN Task t ON p.ProjectId = t.ProjectId
GROUP BY p.ProjectId, p.Name;

-- 视图创建后，可将其用作数据源
SELECT * FROM ProjectNameWithMinTaskId;
```

```sql
-- 15-7 使用视图的复杂查询
SELECT 
    pt.ProjectName,
    pt.MinTaskId TaskId,
    t.Title
FROM Task t
INNER JOIN ProjectNameWithMinTaskId pt -- 就像给表设定别名一样
ON t.TaskId = pt.MinTaskId;
```

视图的优点：
- 封装复杂的连接可以减少代码的复杂度，提高代码重用性；
- 视图可以与表分开进行安全设置（如，可以让用户访问视图，而不是底层的基础表）；
- 视图可以限制某些用户显示的列和行。

视图的缺点：
- 在 MySQL 中，视图并不是一个永久的结构。虽然视图的定义始终可用，但视图中的内容是临时的。即在访问视图时，里面的数据都是现场进行检索的，对视图使用相同的查询，每次得到的结果可能是不同的；
- 视图仅仅看起来很简单，并不意味着底层数据模型也很简单。一个简单的结果视图运行起来可能非常消耗资源；
- 因为视图易于理解，所以，开发人员可能会倾向于在视图上构建越来越多的内容。当视图连接到其他视图时，可能会出现严重的性能问题；
- 如果使用的是 MySQL 以外的其他数据库，则需要确认由视图生成的表是否每次都会保持不变，还是每次都会重新创建。

## 理解事务

事务：一个包括多个独立步骤的操作；所有这些步骤都必须成功完成，否则事务本身将无法成功。

### 事务的示例

一个事务是一组操作，这些操作共同构成一个不可分割的集合，即一个事务必须作为一个独立操作成功或失败。发生错误时保证数据的一致性和可恢复性。

事务的执行：
1. 初始化事务
2. 执行操作
3. 执行提交或中止操作
    - 提交（commit）：操作执行成功，并且数据提交成功。
    - 中止（abort）：如果事务中的某个操作失败，那么所有操作都将回滚，并且数据将恢复到其原始状态。

### ACID

一个事务必须同时满足四个属性：
- 原子性（atomicity）：事务中的所有操作要么都成功，要么都失败。当该事务失败时，在该事务中已经完成的操作也都将被取消；
- 一致性（consistency）：事务不应该对数据产生任何不利影响。如果数据在事务开始前是一致的，则在事务结束后也应该是一致的；
- 隔离性（isolation）：事务彼此独立发生，也不会相互干扰。这意味着，两个事务不能同时操作同一数据；
- 持久性（durability）：如果事务成功，则各个操作所做的更改是永久性的。

操作系统同时执行多个事务：必须适当安排事务，使事务按顺序发生。
- 串行化：依次执行多个事务的过程，使事务按顺序发生。效率低下，可能会使资源处于等待状态。
- 交错执行：必须定义一些机制，允许在多个事务交错执行操作的同时，确保数据的一致性。

一个事务可以有多个状态，且不同状态形成一个有限状态机（一种具有有限状数量状态的系统）。
- 激活（Active）：事务在其执行期间处于活动状态，这是任意一个事务的初始状态；
- 部分提交（Partially committed）：在事务执行其最终操作后，被认为已部分提交；
- 已提交（Committed）：一旦事务成功执行其所有操作，且与其他活动事务没有冲突，则操作将永久提交；
- 失败（failed）：当事务的一个操作无法成功完成时，事务将失败；
- 中止（Aborted）：如果事务失败，那么事务管理器将数据回滚到其原始状态，操作将中止。

```sql
-- 15-8 事务示例：在一个事务中执行三个操作，模拟支票账户和储蓄账户之间的典型转账操作
/*
 *  使用 START TRANSACTION 语句构建事务
 *  
 *  在事务中，必须使用 COMMIT 关键字保存 SELECT 和 UPDATE 语句定义的更改。
 *  如果其中一条语句失败，则提交也会失败，并且所有更改都不会保存到数据库中。
 * */
START TRANSACTION;

SELECT balance FROM checking WHERE customer_id=10233276;

UPDATE checking SET balance = balance -200.00
WHERE customer_id=10233276;

UPDATE savings SET balance = balance + 200.00 WHERE
customer_id=10233276;

COMMIT;
```

## schema 优化

### 选择最佳的数据类型

[MySQL 中可用的数据类型的完整列表](https://dev.mysql.com/doc/refman/5.7/en/data-types.html)

### 索引

MySQL 默认使用 InnoDB 存储引擎，它支持主键和外键。

B-树索引：
使用 B-树数据结构存储数据。其中，根节点指向下一个子结点。存储引擎会跟踪这些指针，直到它找到所需的数据为止。树中的每个节点都有一个键及指向子页面和树中下一个叶子节点的指针。B-树索引加速了数据访问，可以很好地通过完整的键值、键范围或键前缀进行数据查找。

[Introduction of B Tree](https://www.geeksforgeeks.org/dsa/introduction-of-b-tree-2/)

```sql
-- 15-9 创建索引
/*
 *  使用姓和名作为键
 *  所有姓和名都相同的人可以很容易找到，所有姓或名相同的人也能很快找到
 *
 *  KEY 子句根据名字、姓氏、出生日期创建一个索引
 *  键中属性的顺序很重要：这个键是先用名字构建的，可加快对名字的查找速度
 * */
CREATE TABLE person (
    lastname VARCHAR(50) NOT NULL,
    firstname VARCHAR(50) NOT NULL,
    dob DATE NOT NULL,
    KEY(firstname, lastname, dob)
);
```

哈希索引：
引用于执行键的精确查找。每个键列都将被使用。用于键的列值被哈希在一起来生成每行的唯一值，然后使用这个哈希值查找数据。

哈希索引的限制：
- 不能用于排序
- 只能用于相等（=），不能用于其他 SQL 运算符（如 IN、LIKE）

其他索引引擎可使用 MySQL 内置函数（如 CRC32）实现哈希索引，允许计算任意一列的哈希值。

[An Overview of MySQL Database Indexing](https://severalnines.com/blog/overview-mysql-database-indexing)

```sql
-- 15-10 使用哈希索引：查找具有特定名字的人（索引从名字列开始，所以查找可能需要三个列）
/*
 *  名字被用作哈希索引；
 *  MEMORY 被用作存储引擎（哈希索引是 MEMORY 引擎的默认索引类型）；
 *  MEMORY 是 MySQL 中唯一支持显式哈希索引的存储引擎
 *
 *  名字被定义为哈希索引，所以 MEMORY 引擎将生成一个哈希表，
 *  每个人的表的每一行都基于名字计算出一个哈希值，这个哈希值用于快速查找数据。
 * */
CREATE TABLE person (
    firstname VARCHAR(50) NOT NULL,
    lastname VARCHAR(50) NOT NULL,
    KEY USING HASH(firstname)
)Engine=MEMORY;
```

## 小结

- 子查询：在一个查询中嵌入其他完整查询，可独立存在，很少需要修改。可以为 IN 运算符提供值，在连接中充当表，或在 SELECT 列表中求得单个值。
- 视图：存储在数据库中的命名的查询。可以隐藏数据模型的复杂性，提供一种易于使用的抽象方式。将导致性能问题的原因隐藏起来。
- 事务：一组操作，操作一起形成一个不可分割的整体，要么成功，要么失败。事务要满足 ACID（原子性、一致性、隔离性、持久性）。
- 优化 MySQL 数据库的方法：
    - 为列选择适当的数据类型
    - 通过索引优化数据查询
- 两种索引类型：
    - B-树索引（B-树数据结构）
    - 哈希索引（哈希表）