Caffeine 是一个高性能的 [Java](Java.md) [缓存](../../Backend/缓存/缓存.md)库，由 Ben Manes 开发，旨在提供接近最优的命中率和并发访问性能。它是 Guava Cache 的现代替代品，在许多基准测试中表现优异。

## 主要特性

1. **高性能**：使用现代并发技术实现高吞吐量和低延迟
2. **灵活的淘汰策略**：
    - 基于大小
    - 基于时间（访问后过期、写入后过期）
    - 基于引用（软引用、弱引用）
3. **自动加载**：当缓存未命中时自动从指定加载器加载值
4. **统计**：内置命中率统计功能
5. **异步API**：支持 CompletableFuture 的异步操作

## 基本用法

```java
// 创建缓存
Cache<String, Object> cache = Caffeine.newBuilder()
    .maximumSize(10_000)
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .build();

// 放入值
cache.put("key1", "value1");

// 获取值（如果不存在返回null）
Object value = cache.getIfPresent("key1");

// 获取或计算（如果不存在）
value = cache.get("key2", k -> computeExpensiveValue(k));
```


## 高级功能

1. **异步缓存**：
```java
AsyncCache<String, Object> asyncCache = Caffeine.newBuilder()
											   .maximumSize(10_000)
											   .buildAsync();

CompletableFuture<Object> future = asyncCache.get("key", k -> asyncComputeValue(k));
```

2. **监听器**：
```java
Cache<String, Object> cache = Caffeine.newBuilder()
									 .removalListener((String key, Object value, RemovalCause cause) -> 
											System.out.printf("Key %s was removed (%s)%n", key, cause))
								     .build();
```

3. **刷新策略**：
```java
LoadingCache<String, Object> cache = Caffeine.newBuilder()
										    .refreshAfterWrite(1, TimeUnit.MINUTES)
										    .build(this::computeExpensiveValue);
```