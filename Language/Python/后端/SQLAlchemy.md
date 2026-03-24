SQLAlchemy 是 Python 中最流行的 SQL 工具包和对象关系映射（ORM）框架。它提供了一套高级的、以 Python 方式与关系型数据库交互的工具。

## 核心概念

1. **ORM（对象关系映射）**：将数据库表映射为 Python 类，将表中的记录映射为类的实例，让你用操作对象的方式操作数据库。
```python
# 定义模型（对应数据库表）
from sqlalchemy import Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    age = Column(Integer)

# 使用对象方式操作
user = User(name="张三", age=25)
session.add(user)
session.commit()
```

2. **Core（核心层）**：更底层的 SQL 抽象，直接执行 SQL 表达式，性能更高。
```python
from sqlalchemy import create_engine, text

engine = create_engine('sqlite:///users.db')
with engine.connect() as conn:
    result = conn.execute(text("SELECT * FROM users WHERE age > :age"), {"age": 18})
    for row in result:
        print(row)
```

## 主要特性

1. **数据库抽象**：支持多种数据库，切换只需更改连接字符串：
	- PostgreSQL
	- MySQL
	- SQLite
	- Oracle
	- Microsoft SQL Server
2. **连接池**：自动管理数据库连接，提高性能
3. **事务管理**：
```python
try:
    user = User(name="李四")
    session.add(user)
    session.commit()  # 提交事务
except:
    session.rollback()  # 回滚事务
```
4. **关系映射**：
```python
class User(Base):
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    posts = relationship("Post", back_populates="author")

class Post(Base):
    __tablename__ = 'posts'
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey('users.id'))
    author = relationship("User", back_populates="posts")
```

## 优势

1. **提高开发效率**：无需写大量 SQL
2. **类型安全**：Python 类型检查
3. **安全性**：自动防止 SQL 注入
4. **可维护性**：代码结构清晰

## 缺点

1. **学习曲线**：相对复杂
2. **性能开销**：ORM 层有性能损耗
3. **复杂查询**：某些复杂 SQL 不如原生 SQL 直观