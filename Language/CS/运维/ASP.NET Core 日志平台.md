以 .NET 8 的 ASP.NET Core 服务为例，搭建自建 [ELK](../../../DevOps/Logging/ELK.md) / EFK 日志监控平台。

## 搭建方案

### 第一步：搭建 ELK / EFK 服务

1. [部署 Elasticsearch](../../../DevOps/Logging/Elasticsearch.md#安装)
2. [部署 Filebeat](../../../DevOps/Logging/Filebeat.md#安装)
3. [部署 Kibana](../../../DevOps/Logging/Kibana.md#安装)

### 第二步：配置项目的日志输出

1. **安装必要的 NuGet 包**
```bash
Install-Package Serilog
Install-Package Serilog.Sinks.File
Install-Package Serilog.Sinks.Console
Install-Package Serilog.Formatting.Compact # 推荐：JSON格式
Install-Package Serilog.Enrichers.Process # 可选：丰富上下文
Install-Package Serilog.Enrichers.Thread # 可选
```

2.  **在 `Program.cs` 中配置 `Serilog`**
```cs
using Serilog;

var builder = WebApplication.CreateBuilder(args);

// 配置 Serilog
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information() // 设置全局最低日志级别
    .MinimumLevel.Override("Microsoft.AspNetCore", LogEventLevel.Warning) // 过滤框架噪音
    .Enrich.FromLogContext() // 自动捕获上下文信息
    .Enrich.WithMachineName() // 记录机器名
    .Enrich.WithProcessId() // 记录进程ID
    .WriteTo.Console(
        outputTemplate: "[{Timestamp:HH:mm:ss} {Level:u3}] {Message:lj}{NewLine}{Exception}"
    ) // 开发时查看
    .WriteTo.File(
        path: "Logs/log-.txt", // IIS应用池账号需有此目录写入权限
        rollingInterval: RollingInterval.Day, // 按天分割文件
        retainedFileCountLimit: 30, // 保留30天
        fileSizeLimitBytes: 10 * 1024 * 1024, // 单个文件最大10MB
        rollOnFileSizeLimit: true,
        shared: true, // 允许多进程写入同一文件（IIS相关）
        outputTemplate: "{Timestamp:yyyy-MM-dd HH:mm:ss.fff zzz} [{Level:u3}] {MachineName} {ProcessId} {Message:lj}{NewLine}{Exception}"
    )
    .WriteTo.File(
        new CompactJsonFormatter(), // 为Filebeat收集，输出JSON格式
        "Logs/log-.json",
        rollingInterval: RollingInterval.Day,
        retainedFileCountLimit: 30
    )
    .CreateLogger();

builder.Host.UseSerilog(); // 将Serilog设置为日志提供程序

// ... 其他服务配置

var app = builder.Build();
// ... 中间件管道配置
app.Run();
```

3. **在代码中记录结构化日志**：通过依赖注入使用 `ILogger<T>`
```cs
public class WeatherForecastController : ControllerBase
{
    private readonly ILogger<WeatherForecastController> _logger;

    public WeatherForecastController(ILogger<WeatherForecastController> logger)
    {
        _logger = logger;
    }

    [HttpGet]
    public IEnumerable<WeatherForecast> Get()
    {
        // 记录带有属性的结构化日志（便于在Kibana中筛选分析）
        _logger.LogInformation("获取天气预报数据，请求来源：{ClientIP}", 
            HttpContext.Connection.RemoteIpAddress?.ToString());

        try
        {
            // 你的业务逻辑
            var forecast = Enumerable.Range(1, 5).Select(index =>
                new WeatherForecast
                {
                    Date = DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
                    TemperatureC = Random.Shared.Next(-20, 55)
                })
            .ToArray();

            _logger.LogDebug("生成了 {ForecastCount} 条预报数据", forecast.Length);
            return forecast;
        }
        catch (Exception ex)
        {
            // 记录异常时，包含完整的异常对象和上下文
            _logger.LogError(ex, "处理获取天气预报请求时发生错误，用户：{UserId}", 
                User.Identity?.Name ?? "anonymous");
            throw;
        }
    }
}
```



### 第三步：在 Kibana 中查看与分析日志

- **访问 Kibana**：打开浏览器，访问 `http://localhost:5601`
- **创建索引模式**：进入 **Stack Management** > **索引模式** > **创建索引模式**，输入 `dotnet-api-logs-*`，选择 `@timestamp` 字段作为时间字段。
- **发现日志**：进入 **Analytics** > **Discover**，选择你创建的索引模式，即可搜索、筛选和查看所有日志。
- **创建可视化与仪表板**：
	- 进入 **Analytics > Dashboard** 创建新的仪表板。
	- 点击 “创建可视化”，选择如 **TSVB (时间序列可视化构建器)** 或 **Lens**。
	- 可以创建以下有用的图表：
		- **请求量/错误率趋势图**（基于时间）
		- **HTTP状态码分布**（饼图）
		- **慢请求追踪**（筛选 `Duration` 大于某阈值的日志）
		- **异常类型统计**（筛选 `Level: Error` 并聚合 `Exception` 字段）