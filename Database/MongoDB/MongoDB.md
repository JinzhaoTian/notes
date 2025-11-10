[MongoDB](https://www.mongodb.com/) 是一个开源的 NoSQL 文档数据库，采用文档导向的数据模型，属于非关系型数据库（NoSQL，Not only SQL）的一种，由 C++ 语言编写。

MongoDB 支持的数据结构非常松散，是**类似 json 的 bson 格式**，称为文档，因此可以存储比较复杂的数据类型。该数据库最大的特点是它支持的查询语言非常强大，其语法有点类似于面向对象的查询语言，几乎可以实现类似关系数据库单表查询的绝大部分功能，而且还支持对数据建立索引。

**主要特点**

1. **面向集合文档的存储**：适合存储 Bson（json 的扩展）形式的数据
2. **格式自由**，数据格式不固定，生产环境下修改结构都可以不影响程序运行；
3. **强大的查询语句**，面向对象的查询语言，基本覆盖 sql 语言所有能力
4. **完整的索引支持**，支持查询计划
5. 支持复制和自动故障转移
6. 支持二进制数据及大型对象（文件）的高效存储
7. 使用**分片集群**提升系统扩展性
8. 使用**内存映射存储引擎**，把磁盘的 IO 操作转换成为内存的操作

**核心概念**

- **数据库(Database)**：一个 MongoDB 实例可以包含多个数据库
- **集合(Collection)**：相当于关系型数据库中的"表"，包含多个文档
- **文档(Document)**：基本数据单元，相当于"行"，采用 BSON 格式
- **字段(Field)**：文档中的键值对，相当于"列"


## Overview

1. [MongoDB 架构](MongoDB%20架构.md)
2. [MongoDB 集群](MongoDB%20集群.md)
3. GUI：
	- [MongoDB Compass](MongoDB%20Compass.md)




## 安装使用

### Docker

```bash
docker pull mongodb/mongodb-community-server:latest
```

```
docker run --name mongodb -p 27017:27017 -d mongodb/mongodb-community-server:latest
```


### Windows

1. [下载 MongoDB 社区版 .msi 安装程序](https://www.mongodb.com/try/download/community)，
2. 双击 .msi 文件，使用默认MSI 安装向导在 Windows 上安装 MongoDB
	- 该向导将引导您完成 MongoDB 和 MongoDB Compass 的安装。





## 连接


### Windows

MongoDB 配置文件如下：
```conf
# mongod.conf

# for documentation of all options, see:
#   http://docs.mongodb.org/manual/reference/configuration-options/

# Where and how to store data.
storage:
  dbPath: C:\Program Files\MongoDB\Server\8.0\data

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path:  C:\Program Files\MongoDB\Server\8.0\log\mongod.log

# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1


#processManagement:

#security:

#operationProfiling:

#replication:

#sharding:

## Enterprise-Only Options:

#auditLog:
```

1. **配置远程访问** 

修改 `network interfaces` ，
```
# network interfaces
net:
  port: 27017
  bindIp: 0.0.0.0   # 允许所有IP连接
```

重启服务，
```
net stop MongoDB
net start MongoDB
```


2. **启用认证（推荐）**

修改 `security`，
```
security:
  authorization: enabled
```

重启服务，
```
net stop MongoDB
net start MongoDB
```





## 创建账户



1. 创建管理员账户，
```sql
use admin
db.createUser({user: "adminUser", pwd: "yourAdminPassword", roles: [{role: "userAdminAnyDatabase"}]})
```


2. 创建远程账户，
```sql
use admin
db.createUser({user: "remoteUser", pwd: "yourSecurePassword", roles: [{role: "readWrite", db: "yourDatabase"}]})


db.createUser({user: "thbimmongodb", pwd: "Psd242asiuFG4Rxyy6546hDFur", roles: [{role: "readWrite", db: "bim"}]})
```




#### 数据库用户角色

- `read` ：提供在所有非系统集合和 `system.js` 集合上读取数据的能力。
- `readWrite` ：提供 read 角色的所有权限，以及在所有非系统集合和 `system.js` 集合上修改数据的能力。


#### 数据库管理角色

- `dbAdmin` ：提供执行管理任务的能力，例如架构相关任务、索引和收集统计信息。该角色不授予用户和角色管理特权。
- `dbOwner` ：数据库所有者可以对数据库执行任何管理操作。此角色结合了 `readWrite`、`dbAdmin` 和 `userAdmin` 角色授予的权限。
- `userAdmin` ：提供在当前数据库上创建和修改角色和用户的能力。由于 `userAdmin` 角色允许用户向任何用户（包括自己）授予任何特权，因此该角色还间接提供对数据库或（如果作用域为 `admin` 数据库）集群的超级用户访问权限。


#### 集群管理角色

- `clusterAdmin` ：提供最大的集群管理访问权限。
- `clusterManager` ：提供对集群的管理和监控操作。
- `clusterMonitor` ：提供对监控工具（例如 MongoDB Cloud Manager 和 Ops Manager监控代理）的只读访问权限。