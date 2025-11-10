
![](_imgs/Pasted%20image%2020230704135235.png)

PostgreSQL是以加州大学伯克利分校计算机系开发的POSTGRES， 版本 4.2为基础的**对象关系型数据库管理系统（ORDBMS）**。POSTGRES 领先的许多概念在很久以后才出现在一些商业数据库系统中。PostgreSQL是最初的伯克利代码的开源继承者。它支持大部分 SQL 标准并且提供了许多现代特性：

- 复杂查询
- 外键
- 触发器
- 可更新视图
- 事务完整性
- 多版本并发控制

同样，PostgreSQL可以用许多方法扩展，比如， 通过增加新的：

- 数据类型
- 函数
- 操作符
- 聚集函数
- 索引方法
- 过程语言

并且，因为自由宽松的许可证，任何人都可以以任何目的免费使用、修改和分发PostgreSQL， 不管是私用、商用还是学术研究目的。经过二十多年的发展，PostgreSQL是世界上可以获得的最先进的开源数据库。



PostgreSQL是开源对象关系型数据库，使用SQL语言来执行资料的查询。

PostgreSQL数据库的进程可以分为三类：

1. 后台进程
2. 服务器进程
3. 客户端进程


## 架构基础

PostgreSQL使用一种客户端/服务器的模型。一次PostgreSQL会话由下列相关的进程（程序）组成：

- 一个服务器进程，它管理数据库文件、接受来自客户端应用与数据库的联接并且代表客户端在数据库上执行操作。 该数据库服务器程序叫做`postgres`。
- 客户端（前端）应用。 客户端应用可能本身就是多种多样的：可以是一个面向文本的工具， 也可以是一个图形界面的应用，或者是一个通过访问数据库来显示网页的网页服务器，或者是一个特制的数据库管理工具。 一些客户端应用是和 PostgreSQL发布一起提供的，但绝大部分是用户开发的。

和典型的客户端/服务器应用（C/S应用）一样，这些客户端和服务器可以在不同的主机上。 这时它们通过 TCP/IP 网络联接通讯。 你应该记住的是，在客户机上可以访问的文件未必能够在数据库服务器机器上访问（或者只能用不同的文件名进行访问）。主服务器进程总是在运行并等待着客户端联接， 而客户端和相关联的服务器进程则是起起停停。



## 常用命令

1. ` \?`：查看psql命令列表
2. `\h`：查看SQL命令的解释，比如`\h select`
3. `\l`：列出所有数据库
4. `\du`：列出所有用户
5. `\d`：列出当前数据库的所有表格
6. ` \d [table_name]`：列出某一张表格的结构
7. ` \c [database_name]`：连接其他数据库
8. `\password`：设置当前登录用户的密码
9. `\password [user]`：修改用户密码
10. `\q`：退出



## 登陆控制台指令


1. 创建`[username]`用户：`CREATE USER [username] WITH PASSWORD [password]`
2. 删除数据库用户：`drop user [username];`



1. 创建`dbname`数据库：`create database [dbname];`
2. 删除数据库：`drop database [dbname];`



1. 创建数据库表：`CREATE TABLE COMPANY( ID INT PRIMARY KEY NOT NULL, NAME TEXT NOT NULL, AGE INT NOT NULL, ADDRESS CHAR(50), SALARY REAL);`

2. 删除数据库表： `drop table company;`



1. 删除行：`DELETE FROM [table name] WHERE [column name] = '[column name]';`


## 安装

### Windows 安装

1. 下载，官网：[PostgreSQL: The world's most advanced open source database](https://www.postgresql.org/)
2. 对于 Windows 上的安装，下载安装程序exe之后，双击打开
![](_imgs/Pasted%20image%2020240126164051.png)
![](_imgs/Pasted%20image%2020240126164102.png)
![](_imgs/Pasted%20image%2020240126164116.png)



### Mac 安装

在mac上用homebrew安装：

```
brew install postgresql
```

安装完成后可以查看版本：

```
psql --version
```

初始化数据库：

```
initdb /usr/local/var/postgres
```

### 启动服务

启动服务：

```
pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log start
```

关闭服务：

```
pg_ctl -D /usr/local/var/postgres stop -s -m fast
```

设置开机自启动：

```
ln -sfv /usr/local/opt/postgresql/*.plist ~/Library/LaunchAgents launchctl load ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist
```



### 创建数据库和账户

mac安装PostgreSQL后不会创建用户名数据库，所以首先要执行命令：

```
createdb
```

然后就可以登陆命令行界面：

```
psql
```

