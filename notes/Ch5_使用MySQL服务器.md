# 使用 MySQL服务器

目标：

- 安装 MySQL 数据库服务器和客户端；
- 使用命令行接口连接 MySQL 服务器；
- 安装 MySQL Workbench 数据库管理应用程序；
- 确定主要的 MySQL Workbench 元素；
- 使用 Schemas 面板查看数据库的详细信息；
- 使用 SQL 脚本创建数据库；
- 执行 SELECT 查询。

## MySQL 安装

**记住 root 密码**

1. 下载文件：[MySQL8 下载页面](https://dev.mysql.com/downloads/windows/installer/)
2. 跳过登录：单击 Login 按钮下的 "No thanks, just start my download" 链接。
3. 开始安装：系统提示选择安装类型时，在左列选择 Developer Default 选项。
4. 工具选择：MySQL 工具列表，查看要安装的工具。列表中应包括 MySQL Server 和 MySQL Workbench 以及 用于多种语言的连接器、当前版本 MySQL 文档、可用于 MySQL 练习的样本文件。单击 Execute 按钮并等待。
5. 产品配置：大多数情况，选择默认选项即可。几个特殊的配置选项：
   - High Availability 窗口：使用默认选项 Standalone MySQL Server/Classic MySQL Replication。
   - Type and Networking 窗口：使用默认选项 Development Computer With TCP/IP，使用端口号 3306，并打开 Windows 防火墙端口。
   - Authenticaiton Method 窗口：使用选项 Legacy Authentication method。
   - Account and Roles 窗口：需要为 root 账户定义个密码。root 用户允许用户在数据库中执行任何操作。
6. MySQL Router 设置：用于数据库集群，此处不做任何修改。

## MySQL Notifier

下载文件：[MySQL Notifier](https://downloads.mysql.com/archives/notifier)

## 命令行接口

打开 MySQL 命令行客户端，根据提示输入密码，将看到一个命令窗口，出现 "mysql>" 提示符。
窗口中输入命令：

```sql
show databases;
```

## MySQL Workbench 入门

## 小结
- 如何设置MySQL，访问MySQL的工具（服务器本身、Notifier、命令行接口）
- MySQL Workbench：
   - 直观地探索数据库及其结构
   - 编写 SQL
   - 执行 SQL
   - 获取和探索数据
   - 查询统计信息
   
## 本课练习

### 练习 5-1：运行工具
如果你在学习本课时没有设置这些工具，那么你的第一个练习是下载并安装这些工具。同时，你还应该运行本课中介绍的命令。

### 练习 5-2：列出城市
在 MySQL Workbench 中，单击窗口左侧的模式列表中的 world。这将选择要使用的模式，并在窗格中显示其中的表。在选择 world 后，在 Query 1 面板中输入以下代码行，用它替换已经存在的任何文本：

```sql
select * from city limit 0,100;
```

在执行这行代码时，结果面板中会显示什么？
对代码行做如下修改，然后再次运行。这会如何改变输出结果呢？

```sql
select * from city order by Name limit 0,100;
```

### 练习 5-3：寻找人口较少的城市
在练习 5-2 中，你应该看到以两种不同的方式显示的包括 100 个城市的列表。输入以下代码，并观察它会如何改变输出：

```sql
select Name, CountryCode, Population
from city
where Population < 10000
order by Population;
```

在执行这行代码时，结果面板中会显示什么？改变测试来寻找人口较多的城市。