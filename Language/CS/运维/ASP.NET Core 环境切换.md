在 ASP.NET Core 中，根据不同的环境（如开发、测试、生产）自动切换 `appsettings.json` 文件是框架的核心功能，整个过程依赖一个名为 `ASPNETCORE_ENVIRONMENT` 的环境变量来决定当前运行环境，并自动加载对应的配置文件。

## 环境设置

1. **在开发期间设置**：本地开发时，通常通过项目的 `launchSettings.json` 文件来设置，该文件位于项目的 `Properties` 文件夹下。你可以在其中的 `profiles` 部分，为不同的启动配置（如使用 Kestrel 或 IIS Express ）定义不同的环境变量。
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

4. **部署到 IIS 时，通过 IIS 管理器设置**：![](_imgs/Pasted%20image%2020251209134638.png)![](_imgs/Pasted%20image%2020251209135228.png)![](_imgs/Pasted%20image%2020251209135455.png)
	- 在集合编辑器中，点击 **“添加”**，并设置：
	    - **name（名称）**：输入 `ASPNETCORE_ENVIRONMENT`
	    - **value（值）**：根据环境需要，输入 `Development`、`Test`、`Production` 或其他自定义环境名
	- 依次点击 **“确定”**、**“应用”**，然后**重启**该网站对应的应用程序池或网站本身，使设置生效。

5. **部署到 IIS 时，修改 `web.config` 文件**：
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath="dotnet" arguments=".\YourApp.dll" stdoutLogEnabled="false" stdoutLogFile=".\logs\stdout">
        <!-- 在此添加环境变量 -->
        <environmentVariables>
          <!-- 设置环境为 Test -->
          <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Test" />
          <!-- 可以在此添加其他应用所需的环境变量 -->
        </environmentVariables>
      </aspNetCore>
    </system.webServer>
  </location>
</configuration>
```