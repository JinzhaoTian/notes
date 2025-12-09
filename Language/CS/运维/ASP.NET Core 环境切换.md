在 ASP.NET Core 中，根据不同的环境（如开发、测试、生产）自动切换 `appsettings.json` 文件是框架的核心功能，整个过程依赖一个名为 `ASPNETCORE_ENVIRONMENT` 的环境变量来决定当前运行环境，并自动加载对应的配置文件。

## 环境设置

1. **在开发期间设置**：本地开发时，通常通过项目的 `launchSettings.json` 文件来设置，该文件位于项目的 `Properties` 文件夹下。你可以在其中的 `profiles` 部分，为不同的启动配置（如使用Kestrel或IIS Express）定义不同的环境变量。
```json
"profiles": {
  "MyApp.Development": {
    "commandName": "Project",
    "environmentVariables": {
      "ASPNETCORE_ENVIRONMENT": "Development"
    }
  },
  "MyApp.Test": {
    "commandName": "Project",
    "environmentVariables": {
      "ASPNETCORE_ENVIRONMENT": "Test"
    }
  }
}
```

2. **在部署环境中设置**：在测试、预发布或生产服务器上，需要直接设置操作系统的环境变量。
	- **Windows**：`set ASPNETCORE_ENVIRONMENT=Test`
	- **Linux/macOS**：`export ASPNETCORE_ENVIRONMENT=Test`

3. **使用命令行启动**：无论何时，都可以在启动应用时通过命令行参数直接指定环境
```bash
dotnet run --environment Production
```

