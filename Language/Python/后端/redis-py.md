redis-py 是 Redis 官方的 Python 客户端库，它提供了 Python 程序与 Redis 数据库进行交互的完整接口。

## 核心功能

redis-py 实现了 Redis 的所有核心功能：

1. **基本数据类型操作**
    - 字符串（String）：`set()`、`get()`
    - 哈希（Hash）：`hset()`、`hgetall()`
    - 列表（List）：`lpush()`、`rpop()`
    - 集合（Set）：`sadd()`、`smembers()`
    - 有序集合（Sorted Set）：`zadd()`、`zrange()`
2. **高级功能**
    - 连接池管理
    - 管道（Pipeline）：批量执行命令
    - 事务（Transaction）：通过 `WATCH`/`MULTI`/`EXEC` 实现
    - 发布订阅（Pub/Sub）：消息队列模式
    - Lua 脚本：支持脚本执行和缓存
    - 集群支持：Redis Cluster 模式
    - 哨兵支持：Redis Sentinel 高可用模式
3. **编程模式**
    - **同步模式**：传统的阻塞式调用
    - **异步模式**：从 4.0 版本开始支持 `async/await` 语法

## 使用示例

```python
import redis

# 连接 Redis
client = redis.Redis(
    host='localhost', 
    port=6379, 
    db=0,
    decode_responses=True  # 自动将响应解码为字符串
)

# 基本操作
client.set('name', 'Redis')
print(client.get('name'))  # 输出: Redis

# 哈希操作
client.hset('user:1', mapping={'name': 'Alice', 'age': 25})
print(client.hgetall('user:1'))  # 输出: {'name': 'Alice', 'age': '25'}

# 使用连接池
pool = redis.ConnectionPool(host='localhost', port=6379, db=0)
client = redis.Redis(connection_pool=pool)

# 管道批量操作
with client.pipeline() as pipe:
    pipe.set('key1', 'value1')
    pipe.set('key2', 'value2')
    pipe.incr('counter')
    results = pipe.execute()
```

## 异步版本

从 redis-py 4.0 开始，内置了异步支持：
```python
import asyncio
from redis.asyncio import Redis

async def main():
    client = Redis(host='localhost', decode_responses=True)
    
    await client.set('async_key', 'async_value')
    value = await client.get('async_key')
    print(value)  # 输出: async_value
    
    await client.aclose()

asyncio.run(main())
```

## 性能优化

可以通过安装 `hiredis` 解析器来提升性能：
```bash
pip install "redis[hiredis]"
```

hiredis 是用 C 语言编写的 Redis 协议解析器，比 Python 原生解析快 10-20 倍。

