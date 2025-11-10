ASP.NET 是 .NET 平台上一个用于生成 Web 应用的热门 Web 开发框架，ASP.NET Core 是运行在 macOS、Linux 和 Windows 上的 ASP.NET 的开放源代码版本。

## 特点

- **模块化设计**：内置依赖注入（DI）、中间件管道、配置管理等企业级特性。
- **高性能**：基于 Kestrel 服务器，性能接近 Go 或 Rust（TechEmpower 基准测试前列）。
- **多范式支持**：
    - **MVC**：适合传统分层架构（类似 Spring MVC）。
    - **Minimal API**：轻量级路由（类似 Express/FastAPI）。
    - **SignalR**：实时双向通信（WebSocket 支持）。
- **深度集成**：
    - 数据库：[Entity Framework Core](Entity%20Framework%20Core.md)（ORM）、Dapper（微ORM）。
    - 安全：Identity（认证/授权）、JWT、OAuth。
    - 微服务：gRPC、Health Checks、分布式缓存。


## 使用

1. **新建项目**
```bash
dotnet new webapi -n TestProject -f net8.0
```

2. **项目结构**
```
TestProgram/
├── TestProject/                        # 项目路径
│   ├── obj/                            # 编译过程中生成的临时文件
│   ├── Properties/                     
│   │   └── appsettings.json            # 应用程序的启动配置
│   ├── Models/                         # 数据
│   ├── DTOs/                           # 数据传输对象
│   ├── Controllers/                    # 控制器
│   ├── Services/                       # 服务
│   ├── appsettings.Development.json    # 开发环境特定的配置文件
│   ├── appsettings.json                # 应用程序级别配置文件
│   ├── TestProject.csproj              # 项目定义文件
│   ├── TestProject.http                # HTTP 请求测试文件
│   └── Program.cs                      # 应用程序入口点
└── TestProgram.sln                     # 解决方案文件
```

### 文件说明

1. `TestProject.csproj`
```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.OpenApi" Version="8.0.16" />
    <PackageReference Include="Swashbuckle.AspNetCore" Version="6.6.2" />
  </ItemGroup>

</Project>
```
- **主要作用**
	- 定义项目类型：Web应用程序（`Microsoft.NET.Sdk.Web`）
	- 设置目标框架：`.NET 8.0`
	- 启用空值检查(Nullable)和隐式using语句
	- 包含两个 NuGet 包依赖：
		- `Microsoft.AspNetCore.OpenApi`：用于实现OpenAPI规范
		- `Swashbuckle.AspNetCore`：用于生成Swagger文档，方便API测试和文档化



2. `Program.cs`
```cs
using TestProject.Services;

namespace TestProject
{
    public class Program
    {
        public static void Main(string[] args)
        {
            var builder = WebApplication.CreateBuilder(args);

            // Add services to the container.
            builder.Services.AddScoped<IWeatherService, WeatherService>();

            builder.Services.AddControllers();
            // Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
            builder.Services.AddEndpointsApiExplorer();
            builder.Services.AddSwaggerGen();

            var app = builder.Build();

            // Configure the HTTP request pipeline.
            if (app.Environment.IsDevelopment())
            {
                app.UseSwagger();
                app.UseSwaggerUI();
            }

            app.UseHttpsRedirection();

            app.UseAuthorization();

            app.MapControllers();

            app.Run();
        }
    }
}

```
- **主要功能**
	- 创建应用程序构建器
	- 配置服务容器
		- 服务注册
		- 添加 Swagger 文档生成
		- 添加 API Explorer
	- 构建应用程序实例
	- 配置 HTTP 请求处理管道：
		- 在开发环境中启用 Swagger 文档 UI
		- 启用 HTTPS 重定向



### 开发说明




## 核心架构


### 依赖注入

ASP.NET Core 内置了一个轻量级的、高性能的依赖注入（Dependency Injection，DI）容器，它是 ASP.NET Core 架构的核心部分，几乎所有组件都通过 DI 来获取它们的依赖项。

#### 服务注册

在 `Startup.cs` 或 `Program.cs` 中注册服务：
```cs
public void ConfigureServices(IServiceCollection services)
{
    // 注册服务
    services.AddTransient<IMyService, MyService>();
    services.AddScoped<IAnotherService, AnotherService>();
    services.AddSingleton<ICacheService, CacheService>();
}
```

##### 注册方式对比

1. **`services.AddTransient<IService, Service>()`**：
	- **用途**：注册 `Service` 作为 `IService` 接口的实现
	- **结果**：服务将以接口类型 `IService` 被解析
	- 典型的"面向接口编程"做法
2. **`services.AddTransient<Service>()`**：
	- **用途**：直接注册 `Service` 类本身
	- **结果**：- 服务将以具体类 `Service` 被解析

| 特性     | `services.AddTransient<IService, Service>()` | `services.AddTransient<Service>()` |
| ------ | -------------------------------------------- | ---------------------------------- |
| 注册类型   | 接口到实现的映射                                     | 具体类                                |
| 解析方式   | 只能通过接口解析                                     | 只能通过具体类解析                          |
| 是否支持接口 | 是                                            | 否(除非类实现了接口)                        |
| 松耦合程度  | 高(依赖抽象)                                      | 低(依赖具体实现)                          |
| 测试友好性  | 高(容易mock)                                    | 低(难以mock)                          |
| 多实现支持  | 可以注册多个实现到同一接口                                | 每个具体类独立注册                          |



#### 生命周期

1. **Transient** ：（瞬时）每次请求都创建新实例
2. **Scoped** ：（作用域）每个 HTTP 请求范围内共享一个实例
3. **Singleton** ：（单例）整个应用生命周期只创建一个实例，适合全局共享的服务，如配置、缓存


#### 服务解析

可以通过以下方式获取服务实例：

1. **构造函数注入**
```cs
public class HomeController : Controller
{
    private readonly IMyService _myService;
    
    public HomeController(IMyService myService)
    {
        _myService = myService;
    }
}
```

2. **从HttpContext手动解析**
```cs
var service = HttpContext.RequestServices.GetService<IMyService>();
```

3. **方法注入**（使用`[FromServices]`特性）
```cs
public IActionResult Index([FromServices] IMyService myService)
{
    // 使用myService
}
```

#### 高级特性

##### 泛型服务注册

```cs
services.AddSingleton(typeof(IRepository<>), typeof(Repository<>))
```

##### 多实现注册

为同一接口注册了多个实现：
```cs
services.AddTransient<IMessageService, EmailService>();
services.AddTransient<IMessageService, SmsService>();

// 在需要的地方注入
public class NotificationController : Controller
{
    private readonly IEnumerable<IMessageService> _messageServices;
    
    public NotificationController(IEnumerable<IMessageService> messageServices)
    {
        _messageServices = messageServices;
    }
    
    public IActionResult SendAll()
    {
        foreach(var service in _messageServices)
        {
            service.Send("Hello");
        }
        return Ok();
    }
}
```

可以使用**工厂委托**正确解析出想要的特定服务：
```cs
// 注册时指定名称
services.AddTransient<EmailService>();
services.AddTransient<SmsService>();

services.AddTransient<Func<string, IMessageService>>(serviceProvider => key =>
{
    return key switch
    {
        "Email" => serviceProvider.GetService<EmailService>(),
        "SMS" => serviceProvider.GetService<SmsService>(),
        _ => throw new KeyNotFoundException()
    };
});

// 使用
public class NotificationController : Controller
{
    private readonly Func<string, IMessageService> _messageServiceFactory;
    
    public NotificationController(Func<string, IMessageService> messageServiceFactory)
    {
        _messageServiceFactory = messageServiceFactory;
    }
    
    public IActionResult SendEmail()
    {
        var emailService = _messageServiceFactory("Email");
        emailService.Send("Hello via Email");
        return Ok();
    }
}
```


##### 工厂模式注册

使用委托控制实例创建：
```cs
services.AddTransient<IService>(provider => 
{
    var otherService = provider.GetService<IOtherService>();
    return new ServiceImpl(otherService);
});
```


##### 选项模式

结合 `IOptions<T>` 使用配置：
```cs
services.Configure<MyOptions>(Configuration.GetSection("MySection"));
```


##### 第三方容器集成

如 Autofac、DryIoc 等



### 路由系统

ASP.NET Core 的路由系统负责将传入的 HTTP 请求映射到相应的处理程序（通常是控制器中的操作方法）。

**主要形式**
- **Conventional Routing** ：基于约定的路由，在启动时全局定义的路由模板
- **Attribute Routing** ：属性路由，通过特性（Attribute）直接在控制器和操作上定义的路由

#### 属性路由

在控制器或操作方法上使用 `[Route]` 特性：
```cs
[Route("products")]
public class ProductsController : Controller
{
    [Route("")]
    public IActionResult Index() { /*...*/ }
    
    [Route("{id}")]
    public IActionResult Details(int id) { /*...*/ }
}
```
- **常用路由属性**
	- `[Route]`：定义路由模板
	- `[HttpGet]`, `[HttpPost]` 等：指定 HTTP 方法
	- `[Area]`：定义区域路由



### 中间件

中间件（Middleware）是 ASP.NET Core 处理 HTTP 请求和响应的核心组件，它们组成一个管道（pipeline），每个中间件都可以处理传入的请求并选择是否将请求传递给下一个中间件。

```
Request -> Middleware1 -> Middleware2 -> ... -> MiddlewareN -> Response
```

#### 内置中间件

ASP.NET Core提供了许多内置中间件：

|中间件|描述|常用方法|
|---|---|---|
|静态文件|提供静态文件支持|`UseStaticFiles()`|
|路由|路由请求到处理程序|`UseRouting()`|
|认证|提供认证支持|`UseAuthentication()`|
|授权|提供授权支持|`UseAuthorization()`|
|CORS|跨域资源共享支持|`UseCors()`|
|会话|会话状态支持|`UseSession()`|
|端点|执行端点路由|`UseEndpoints()`|
|异常处理|处理异常|`UseExceptionHandler()`|
|HTTPS重定向|重定向到HTTPS|`UseHttpsRedirection()`|


### MVC 模式

MVC（Model-View-Controller）模式是一种软件架构设计模式，将应用程序分成三个主要组件：
- **模型**（Model）
	- 负责应用程序的数据和业务逻辑
	- 包含应用程序的状态和业务规则
	- 不依赖于视图和控制器，可以独立测试和开发
- **视图**（View）
	- 负责展示内容和用户界面
	- 使用Razor视图引擎在HTML中嵌入.NET代码
	- 应该包含最少的逻辑，主要关注内容的展示
- **控制器**（Controller）
	- 处理用户交互和请求
	- 使用模型处理数据和业务逻辑
	- 选择要呈现的视图
	- 是请求的入口点，控制应用如何响应请求

这种模式在 ASP.NET Core 中被广泛应用，帮助开发者构建结构清晰、易于维护的 Web 应用程序。

