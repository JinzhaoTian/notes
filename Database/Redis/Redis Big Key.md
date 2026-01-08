
Big Key 通常**指 Key 对应的 Value 很大**：
1. `string` 类型：值大小 > 10KB
2. `list` 类型：列表数量 > 20000 个
3. `zset` 类型：成员数量 > 10000 个
4. `hash` 类型：成员数量只有 1000 个，但这些成员的 value 总大小 > 100MB

## 引发问题

1. **性能问题**：读写耗时高，阻塞 Redis 单线程
2. **内存不均衡**：导致集群数据倾斜
3. **阻塞风险**：DEL 删除可能长时间阻塞服务
4. **网络拥塞**：单次传输数据量过大
5. **持久化问题**：AOF 重写、RDB 生成慢

## 检测方法

```bash
# 1. 使用redis-cli --bigkeys命令
redis-cli --bigkeys

# 2. 使用memory usage命令（Redis 4.0+）
redis-cli MEMORY USAGE your_key

# 3. 使用rdb-tools分析RDB文件
rdb -c memory dump.rdb --bytes 10240 | head

# 4. 使用scan命令扫描
redis-cli --scan --pattern '*' | while read key; do
  echo "Key: $key, Size: $(redis-cli memory usage $key)"
done | sort -nrk4 | head -20
```


## 优化方案

1. **拆分大 Key**：
```python
# 原始大Hash
big_hash = {
    "user:1000:data": "超大value..."
}

# 拆分为多个小Key
# 方法1：分片存储
shard_keys = {
    "user:1000:data:shard1": "第一部分数据",
    "user:1000:data:shard2": "第二部分数据"
}

# 方法2：按业务拆分
user_basic = "user:1000:basic"      # 基本信息
user_contact = "user:1000:contact"  # 联系信息
user_profile = "user:1000:profile"  # 个人资料
```

2. **压缩存储**：
```python
import zlib
import json

# 写入时压缩
data = {"large": "data" * 1000}
compressed = zlib.compress(json.dumps(data).encode())
redis.set("key", compressed)

# 读取时解压
compressed = redis.get("key")
data = json.loads(zlib.decompress(compressed))
```

3. **使用合适的数据结构**：
```python
# 不推荐：String存储JSON数组
redis.set("user:list", json.dumps([user1, user2, ...]))

# 推荐：使用List或Hash存储
for user in users:
    redis.hset(f"user:{user['id']}", mapping=user)
```

4. **分批次操作**：
```python
# 批量删除大Key（避免阻塞）
def delete_big_key(key, batch_size=100):
    cursor = '0'
    while cursor != 0:
        cursor, data = redis.hscan(key, cursor=cursor, count=batch_size)
        if data:
            fields = list(data.keys())
            redis.hdel(key, *fields)
    redis.delete(key)

# 分页读取
def scan_big_hash(key, pattern="*", count=100):
    cursor = '0'
    while cursor != 0:
        cursor, data = redis.hscan(key, cursor=cursor, 
                                  match=pattern, count=count)
        yield from data.items()
```


## 特殊场景处理

1. 场景1：大 Key 删除
```bash
# 异步删除（Redis 4.0+）
UNLINK key_name  # 非阻塞删除

# 渐进式删除
redis-cli --eval del_big_key.lua key_name , 100
```

2. 场景2：热点大Key
```python
# 使用本地缓存+Redis多级缓存
local_cache = {}

def get_hot_data(key):
    # 1. 查本地缓存
    if key in local_cache:
        return local_cache[key]
    
    # 2. 查Redis（只取部分数据）
    data = redis.hgetall(key, count=100)
    
    # 3. 更新本地缓存
    local_cache[key] = data
    return data
```