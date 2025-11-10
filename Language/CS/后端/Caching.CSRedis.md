`Caching.CSRedis` 包含两个核心组件，通常配合使用：
1. **`CSRedisCore`**：这是一个功能强大且活跃维护的 Redis 客户端，提供了全面的 Redis 命令支持，它的 API 设计与 Redis 原生命令高度一致，学习成本低。
2. **`Caching.CSRedis`**：`Microsoft.Extensions.Caching.Distributed.IDistributedCache` 接口的 `CSRedisCore` 实现，很方便地利用它将 `CSRedisCore` 集成到 .NET 8 的依赖注入框架中，作为分布式缓存使用。



## 使用示例

1. **安装**：
```bash
Install-Package CSRedisCore
Install-Package Caching.CSRedis
```

2. **方式一**：**依赖注入（推荐）**：
	- 在 `Program.cs` 中配置，将 `CSRedisClient` 注册为单例服务，然后使用 `Caching.CSRedis` 实现 `IDistributedCache` 接口
```cs
var builder = WebApplication.CreateBuilder(args);

// 配置 Redis 连接字符串
var redisConnectionString = "127.0.0.1:6379,password=YourPassword,defaultDatabase=0";

// 初始化 CSRedisClient 并注册为单例
var csredis = new CSRedisClient(redisConnectionString);
builder.Services.AddSingleton(csredis);

// 使用 Caching.CSRedis 实现 IDistributedCache
builder.Services.AddSingleton<IDistributedCache>(new CSRedisCache(csredis));

// 其他服务配置...
var app = builder.Build();
```

```cs
public class WeatherForecastController : ControllerBase
{
    private readonly IDistributedCache _distributedCache;

    public WeatherForecastController(IDistributedCache distributedCache)
    {
        _distributedCache = distributedCache;
    }

    [HttpGet]
    public string Get(string key)
    {
        // 使用 _distributedCache 进行缓存操作
        var cache = _distributedCache.Get(key);
        if (cache != null)
        {
	        return Encoding.UTF8.GetString(cache);
		}
	    
		// 缓存不存在，从数据库或其他来源获取数据
		var data = "从数据库查询的数据";
		
		// 存储到缓存，设置过期时间
		var options = new DistributedCacheEntryOptions()
							.SetAbsoluteExpiration(TimeSpan.FromMinutes(10));
		_distributedCache.Set(key, Encoding.UTF8.GetBytes(data), options);
		
		return data;
    }
}
```

3. **方式二**：**RedisHelper 静态类**
	- 通过一个静态帮助类 `RedisHelper` 提供所有方法，使用起来非常直接，适合快速开发或在非依赖注入的环境中使用。
```cs
// 在 Program.cs 中初始化
var redisConnectionString = "127.0.0.1:6379,password=YourPassword,defaultDatabase=0";
RedisHelper.Initialization(new CSRedisClient(redisConnectionString));

// 在代码中任何地方直接使用
public class SomeService
{
    public void DoSomething()
    {
        // 设置缓存
        RedisHelper.Set("test_key", "hello world", 60); // 60秒过期
        // 获取缓存
        var value = RedisHelper.Get("test_key");
        // 删除缓存
        RedisHelper.Del("test_key");
    }
}
```



## 进阶功能

1. **批量移除缓存**：
```cs
public async Task RemoveByPrefixAsync(string prefix)
{
    // 注意：Scan 命令在 Redis 键数量多时可能影响性能，生产环境慎用。
    int nextCursor = 0;
    do
    {
        var scanResult = await RedisHelper.ScanAsync(nextCursor, pattern: $"{prefix}*");
        nextCursor = scanResult.Cursor;
        var keys = scanResult.Items;
        if (keys.Any())
        {
            await RedisHelper.DelAsync(keys);
        }
    } while (nextCursor != 0);
}
```

2. **配置连接字符串参数**：
```
127.0.0.1:6379,password=YourPassword,defaultDatabase=0,poolsize=50,connectTimeout=5000,syncTimeout=10000,preheat=true
```
- **参数说明**：
	- `password`：Redis 密码。
	- `defaultDatabase`：默认数据库。
	- `poolsize`：连接池大小。
	- `connectTimeout`：连接超时（毫秒）。
	- `syncTimeout`：发送/接收超时（毫秒）。
	- `preheat`：是否预热连接。

3. **发布与订阅**：
```cs
// 订阅
RedisHelper.Subscribe(
    ("channel1", msg => Console.WriteLine($"收到消息: {msg.Body}")),
    ("channel2", msg => Console.WriteLine($"另一个频道: {msg.Body}"))
);

// 发布
RedisHelper.Publish("channel1", "这是一条发布的消息");
```