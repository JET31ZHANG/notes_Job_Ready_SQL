## 数据库操作
目标：
- 使用 PyMySQL 将 Python 脚本连接到 MySQL 示例；
- 使用 Python 脚本 MySQL 数据库上执行基本的创建、检索、更新和删除（CRUD）操作，包括：
  - 创建和删除数据库；
  - 创建和删除表；
  - 在表中添加、更新和删除列；
  - 在表中添加、更新、检索和删除数据。

准备：
- 在 Python 中安装了 PyMySQL 包；
- 一个在后台运行的 MySQL 示例；
- 正在运行的 MySQL 示例的用户名和密码。

### 使用 PyMySQL 
### 获取 MySQL 版本
### 创建数据库

```python
# 代码清单2：创建数据库
import pymysql
# 根据本地 MySQL 设置的要求更新连接数据
con = pymysql.connect(host='localhost', user='root', password='admin')
with con:
    cur = con.cursor()
    cur.execute("CREATE DATABASE recordshop;")

print("Database created")
```

### 删除数据库

```python
# 程序清单3：删除名为 recordshop 的数据库
import pymysql
# 根据本地的 MySQL 设置，更新连接数据。
con = pymysql.connect(host='localhost', user='root', password='admin')
with con:
    cur = con.cursor()
    cur.execute("DROP DATABASE recordshop;")

print("Database deleted")
```

### 连接到数据库

```python
# 代码清单 A-4 
# 连接到指定数据库
import pymysql

con = pymysql.connect(host='localhost', user='root', password='admin', db='mysql')
print(con)
```

```python
# 代码清单 A-5 
# 更完整的连接过程
import pymysql

# 根据本地的 MySQL 设置，更新连接数据。
con = pymysql.connect(host='localhost', user='root', password='admin', db='mysql')
with con:
    cur = con.cursor()                  # 创建游标对象
    cur.execute("DROP DATABASE IF EXISTS recordshop;")
    cur.execute("CREATE DATABASE recordshop;")
    cur.close()                         # 关闭与MySQL的连接
print("Database created")

# 根据本地的 MySQL 设置，更新连接数据。
con = pymysql.connect(host='localhost', user='root', password='admin', db='recordshop')
with con:
    cur = con.cursor()
    cur.execute("SELECT DATABASE();")
    for row in cur:
        dbname = row[0]
print("Connected to " + dbname)
```

### 显示所有数据库

```python
# 代码清单 A-6 显示所有数据库
import pymysql
# 根据本地的 MySQL 设置，更新连接数据。
con = pymysql.connect(host='localhost', user='root', password='admin')
with con:
    cur = con.cursor()  # 创建游标对象
    cur.execute("SHOW DATABASE;")
    for row in cur:
        print(row[0])
```

## 表运算

### 创建表

```python
# 代码清单 A-7 创建 artist 表
import pymysql
create_table_query = """
                    CREATE TABLE artist (
                    artist_id int(11) NOT NULL AUTO_INCREMENT,
                    fname varchar(40) NOT NULL,
                    lname varchar(40) NOT NULL,
                    isHallOfFame tinyint(1) NOT NULL,
                )   ENGINE=InnoDB DEFAULT CHARSET=latin1;
                """
print(create_table_query)
show_talbe_query = """SHOW TABLES;"""
describe_table_query = """DESCRIBE artist;"""
# 根据本地的 MySQL 设置，更新连接数据。
con = pymysql.connect(host='localhost', user='root', password='admin', db='recordshop')
with con:
    cur = con.cursor()                  # 创建一个游标对象，用于执行 MySQL 查询
    cur.execute(create_table_query)
    cur.execute(show_table_query)
    for row in cur:
        print(row[0])
    
    cur.execute(describe_table_query)
    for row in cur:
        print(row)

```

### 更改表

```python
# 代码清单 A-8 对表进行更改
import pymysql 
alter_query_1 = """ALTER TABLE artist 
                ADD PRIMARY KEY (artist_id);"""
alter_query_2 = """ALTER TABLE artist
                MODIFY artist_id int(11) NOT NUL AUTO_INCREMENT, AUTO_INCREMENT=0;"""

describe_table_query = """DESCRIBE artist;"""
# 
con = pymysql.connect(host='localhost', user='root', password='admin', db='recordshop')
with con:
    cur = con.cursor()
    cur.execute(alter_query_1)
    cur.execute(alter_query_2)
    cur.execute(describe_talbe_query)
    for row in cur:
        print(row)
```
### 删除表

```python
# 代码清单 A-9 删除表
import pymysql
drop_query = """DROP TABLE artist;"""
show_table_query = """SHOW TABLES;"""
# 根据本地的 MySQL 设置，更新连接数据
con = pymysql.connect(host='localhost', user='root', password='admin', db='recordshop')
with con:
    cur = con.cursor()
    cur.execute(drop_query)
    cur.execute(show_table_query)
    for row in cur:
        print(row[0])
    
    print("Ready")

```

### 重建表

```python
# 代码清单 A-10 在 recordshop 数据库中重新构建 artlist 表
import pymysql
drop_artist = "DROP TABLE IF EXISTS artist;"
create_artist = """
                CREATE TABLE artist (
                    artist_id int(11) NOT NULL AUTO_INCREMENT,
                    fname varchar(40) NOT NULL,
                    lname varchar(40) NOT NULL,
                    isHallOfFame tinyint(1) NOT NULL,
                    PRIMARY KEY (artist_id)
                )
                ENGINE=InnoDB DEFAULT CHARSET=latin1;
                """

show_tables = """SHOW TABLES;"""
describe_artist = """DESCRIBE artist;"""
# 根据本地的 MySQL 设置，更新连接数据
con = pymysql.connect(host='localhost', user='root', password='admin', db='recordshop')
with con:
    cur = con.cursor()                  # 创建游标对象
    cur.execute(drop_artist)
    cur.execute(create_artist)
    cur.execute(show_tables)
    for row in cur:
        print("Tables in database: \n" + str(row[0]))
    cur.execute(describe_artist)
    print("\nFields in table:")
    for row in cur:
        print(row)

```

## 数据操作：CRUD

### 创建数据

```python
# 代码清单 A-11 在artlist表中创建数据
import pymysql
insert_query = """INSERT INTO artlist (artlist_id, fname, lname, isHallOfFame)
                VALUES 
                    (1, 'John', 'Lennon', 0),
                    (2, 'Paul', 'McCartney', 0),
                    (3, 'George', 'Harrison', 0),
                    (4, 'Ringo', 'Starr', 0),
                    (5, 'Denny', 'Zager', 0),
                    (6, 'Rick', 'Evans', 0),
                    (10, 'Van', 'Morrison', 0),
                    (11, 'Judy', 'Collins', 0),
                    (12, 'Paul', 'Simon', 0),
                    (13, 'Art', 'Garfunkel', 0),
                    (14, 'Brian', 'Wilson', 0),
                    (15, 'Dennis', 'Wilson', 0),
                    (16, 'Carl', 'Wilson', 0),
                    (17, 'Ricky', 'Fataar', 0),
                    (18, 'Blondie', 'Chaplin', 0),
                    (19, 'Jimmy', 'Page', 0),
                    (20, 'Robert', 'Plant', 0),
                    (21, 'John Paul', 'Jones', 0),
                    (22, 'John', 'Bonham', 0),
                    (23, 'Mike ', 'Love', 0),
                    (24, 'Al ', 'Jardine', 0),
                    (25, 'David', 'Marks', 0),
                    (26, 'Bruce ', 'Johnston', 0);"""
                    
view_records = """SELECT *
                FROM artlist
                LIMIT 5;
                """
# 根据本地的 MySQL 设置，更新连接数据
con = pymysql.connect(hody='localhost', user='root', password='admin', db='recordshop')
with con:
    cur = con.cursor()                  # 创建游标对象
    cur.execute(insert_query)           # 执行插入语句
    cur.execute(view_records)
    con.commit()
    for row in cur:
        print(row)
```

### 检索数据

```python
# 代码清单 A-12 从artlist表中检索数据
import pymysql
retrieve_query = """SELECT * 
                FROM artlist;"""

# 根据本地的 MySQL 设置，更新连接数据
con = pymysql.connect(host='localhost', user='root', password='admin', db='recordshop')
with con:
    cur = con.cursor()                  # 创建游标对象
    cur.execute(retrieve_query)         # 执行查询
    for row in cur:
        print(row)
```

### 更新数据

```python
# 代码清单 A-13 从artlist表中更改数据
import pymysql
select_query = """SELECT * 
                FROM artlist
                WHERE lname = 'Lennon';"""

update_query = """UPDATE artlist
                SET isHallOfFame = 1
                WHERE lname = 'Lennon';"""
# 根据本地的 MySQL 设置，更新连接数据
con = pymysql.connect(host='localhost', user='root', password='admin', db='recordshop')
with con:
    cur = con.cursor()                  # 创建游标对象
    cur.execute(select_query)           # 在更改数据之前查看记录
    for row in cur:
        print(row)

cur.execute(update_query)               # 执行更新语句
con.commit()
cur.execute(select_query)               # 在更改数据之后查看记录
for row in cur:
    print(row)

```

### 删除数据

```python
# 代码清单 A-14 在artlist表中删除记录
import pymysql
select_query = """SELECT *
                FROM artlist
                WHERE lname = 'Fataar';"""

update_query = """DELETE FROM artlist
                WHERE lanme = 'Fataar';"""

# 使用适当的值连接到本地的 MySQL 服务器
con = pymysql.connect(host='localhost', user='root', password='admin', db='recordshop')
with con:
    cur = con.cursor()                  # 创建一个游标对象
    cur.execute(select_query)           # 在更改数据之前查看记录
    for row in cur:
        print(row)
    
    cur.execute(update_query)           # 执行更新查询
    con.commit()

    cur.execute(select_query)           # 在更改数据之后查看记录
    for row in cur:
        print(row)
```