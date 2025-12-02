Kestrel 是 ASP.NET Core 默认的跨平台、高性能、开源的 Web 服务器，是 ASP.NET Core 应用程序的核心组件之一，负责直接处理来自客户端的 HTTP 请求。

## 主要特性

1. **跨平台支持**
    - 可以在 Windows、Linux、macOS 上运行，无需依赖其他 Web 服务器（如 IIS、Apache）。
2. **高性能**
    - 基于异步 I/O 和 .NET 的高性能架构设计，能够处理大量并发连接。
    - 针对低内存消耗和高吞吐量优化。
3. **轻量级**
    - 作为应用程序的一部分运行，无需复杂配置。
4. **支持 HTTPS/HTTP/2**
    - 内置支持 HTTPS（需配置证书）和 HTTP/2 协议。
5. **可配置性**
    - 可以通过代码或配置文件自定义端口、协议、请求超时等。

## 工作原理

1. Kestrel 作为独立服务器直接接收 HTTP 请求，处理后再将响应返回给客户端。
2. 在生产环境中，通常会将 Kestrel 与反向代理服务器（如 Nginx、IIS、Apache）结合使用，以提供负载均衡、SSL 终止、静态文件服务等功能。

## 应用场景

1. **独立运行**
    - 开发环境：直接使用 `dotnet run` 启动，Kestrel 监听本地端口（如 `http://localhost:5000`）。

2. **反向代理后方**
    - 生产环境中，通过 Nginx/IIS 将请求转发给 Kestrel，提升安全性和扩展性。

## 配置

ASP.NET Core 默认集成 Kestrel 的过程是自动完成的，你无需手动添加代码。

1. **代码入口**：`var builder = WebApplication.CreateBuilder(args);` 
	- 是 ASP.NET Core 应用的起点，此方法创建了一个 `WebApplicationBuilder` 对象，它内部封装了默认主机配置逻辑。
	- `CreateBuilder` 内部调用 `Host.CreateDefaultBuilder` 和 `ConfigureWebHostDefaults`
		- `ConfigureWebHostDefaults` 方法执行了一系列默认的 Web 主机配置，其中就包括调用 `UseKestrel` 将 Kestrel 设置为默认的 Web 服务器
2. **服务器就绪**：调用 `var app = builder.Build();` 和 `app.Run();`
	- 内嵌了 Kestrel 服务器、并已配置好默认中间件管道和端点的 Web 应用就开始运行了。



在 `Program.cs` 中，可以通过 `ConfigureWebHostDefaults` 自定义 Kestrel：
```cs
var builder = WebApplication.CreateBuilder(args);

// 手动配置 Kestrel
builder.WebHost.ConfigureKestrel(serverOptions =>
{
    // 监听 HTTP 端口 5000
    serverOptions.Listen(IPAddress.Any, 5000);
    
    // 监听 HTTPS 端口 5001
    serverOptions.Listen(IPAddress.Any, 5001, listenOptions =>
    {
        listenOptions.UseHttps("certificate.pfx", "password");
    });
});

var app = builder.Build();
```

