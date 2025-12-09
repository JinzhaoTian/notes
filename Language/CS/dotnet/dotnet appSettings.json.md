`appSettings.json` 是一个用于存储应用程序配置设置的 JSON 格式配置文件，是现代 .NET 应用（如 ASP.NET Core、.NET 5/6/7+ 等）中替代传统 `web.config` 或 `app.config` 的主要配置方式。

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyDb;Trusted_Connection=True;"
  },
  "ApiSettings": {
    "BaseUrl": "https://api.example.com",
    "Timeout": 30
  }
}
```

## 主要作用

1. **存储配置数据**：
	- 数据库连接字符串
	- API 密钥
	- 日志级别
	- 服务端点
	- 等等...
2. **环境分离**：可通过不同文件（如 `appSettings.Development.json`）为不同环境（开发、生产）配置不同值。
3. **灵活绑定**：配置值可直接绑定到 C# 类的属性，便于强类型访问。

## 环境分离

应用加载哪个 `appSettings.json` 是由 `ASPNETCORE_ENVIRONMENT` 环境变量的值决定的，



> [!tiop]