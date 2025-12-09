`launchSettings.json` 是用于配置 .NET/ASP.NET Core 应用程序**本地开发启动选项**的配置文件，其核心作用是**定义本地开发和调试时的启动参数**，如环境变量、URL、启动命令等，通常由 Visual Studio、.NET CLI（`dotnet new`）为特定项目模板（如 Web 应用、Worker Service）自动生成，简单的控制台或类库项目通常没有。

`launchSettings.json` 仅在开发时使用，不会被部署到生产环境，生产环境的配置应使用 `appsettings.json`、环境变量等其他方式。

## 主要功能

`launchSettings.json` 文件中的设置直接控制你按下 `F5` 或运行 `dotnet run` 时的行为：
1. **定义多个启动配置**：可以为不同场景（如开发、测试）配置不同的启动方式。
2. **设置启动命令**：通过 `commandName` 指定是直接启动项目，还是使用 IIS Express
3. **配置环境变量**：可以预设环境变量，例如最常见的` ASPNETCORE_ENVIRONMENT` 变量，用于指示应用运行在开发（Development）模式。
4. **控制应用 URL 和浏览器**：可以设定应用启动时监听的端口和地址（`applicationUrl`），以及是否自动打开浏览器（`launchBrowser`)。

## 字段含义

```json
{
  "$schema": "http://json.schemastore.org/launchsettings.json",
  "iisSettings": {
    "windowsAuthentication": false,
    "anonymousAuthentication": true,
    "iisExpress": {
      "applicationUrl": "http://localhost:30246",
      "sslPort": 44367
    }
  },
  "profiles": {
    "http": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "launchUrl": "swagger",
      "applicationUrl": "http://localhost:5265",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "https": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "launchUrl": "swagger",
      "applicationUrl": "https://localhost:7012;http://localhost:5265",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "IIS Express": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "launchUrl": "swagger",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

1. **`$schema`**：指向描述文件结构的 JSON 模式定义文件，为编辑提供智能提示和验证。
2. **`profiles`**：启动配置文件
	- **`commandName`**：
		- **用途**：决定启动方式
		- **值**：
			- `"Project"`：直接启动（使用 Kestrel 服务器）
			- `"IISExpress"`：通过 IIS Express 启动。
			- `"IIS"`：通过完整 IIS 启动。
	- **`applicationUrl`**：
		- **用途**：应用启动后监听的 URL 地址，多个地址用分号分隔。
	- **`launchBrowser`**：
		- **用途**：启动后是否自动打开浏览器，默认为 `true`。
		- **值**：
			- `true`：自动打开浏览器
			- `false`：不打开浏览器
	- **`launchUrl`**：
		- **用途**：`launchBrowser` 为 `true` 时，浏览器打开的初始页面路径。
		- **值**：
			- 指定路由路径
	- **`environmentVariables`**：
		- **用途**：为此启动配置设置的环境变量，会覆盖系统环境变量，默认值为 `Production`。
		- **值**：
			- `Development`
			- `Staging`
			- `Production`
			-  自定义（如 `Test`）
	- **`dotnetRunMessages`**：
		- **用途**：启动时是否在控制台显示 .NET 运行时信息
		- **值**：
			- `true`：在命令行启动时输出详细日志。
			- `false`：不输出

> [!tip] IIS Express
> IIS Express 是微软提供的开发人员本地调试 Web 服务器，用于托管 ASP.NET Core 应用，集成在 Visual Studio 中。
> 
> IIS 是正式的生产环境 Web 服务器，是 Windows Server 的功能组件。

