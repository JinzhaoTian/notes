在 OceanBase 数据库中运行建库 SQL 语句的步骤与在 MySQL 中类似。您可以使用 OceanBase 的 `obclient` 客户端或兼容 MySQL 的其他客户端工具（如 `mysql` 命令行工具）连接到 OceanBase 实例，然后执行 SQL 语句。


## 使用 `obclient` 工具运行 SQL 语句

首先连接上数据库，
```bash
obclient -h127.0.0.1 -P2881 -uroot -p
```

然后可以执行 SQL 语句，
```sql
CREATE DATABASE InstructionDB;

USE InstructionDB;

CREATE TABLE IF NOT EXISTS users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  username VARCHAR(50) NOT NULL,
  email VARCHAR(100) NOT NULL,
  password VARCHAR(100) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 其他表结构
```


## 使用 `obclient` 工具执行 SQL 脚本

```bash
obclient -h127.0.0.1 -P2881 -uroot -p < /path/to/schema.sql
```

## `obclient` 使用命令

1. 查看数据库
```
SHOW DATABASES;
```
