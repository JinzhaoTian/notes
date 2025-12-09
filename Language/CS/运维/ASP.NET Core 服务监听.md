

## 服务监听地址设置方式

1. **开发配置文件**：`launchSettings.json` 的 `applicationUrl`
	- 仅限本地开发
2. **代码硬编码**：`UseUrls()` 方法
3. **命令行参数**：`dotnet run --urls="http://*:8080"`
	- 临时测试、命令行快速指定
 4. **环境变量**：`ASPNETCORE_URLS`
	- 容器、PaaS 平台（如 Azure App Service）、服务器环境
 5. **Kestrel 终结点配置**：`appsettings.json` 的 `Kestrel:Endpoints` 节
	- 生产、Docker、需要精细控制（证书、协议）
 6. **IIS 托管**：端口和地址的配置控制权发生了根本性转移，.NET Core 应用（Kestrel 服务器）并不直接对外服务。


**服务监听地址设置方式的优先级如下**：**`Kestrel:Endpoints` 配置** > **环境变量** > **命令行** > **代码** > **开发配置**。


其中，配置逐级覆盖的链条如下：
```mermaid
flowchart LR
    A[应用启动] --> B{检查方式1<br>Kestrel:Endpoints配置}
    B -- 已配置 --> C[以此为准<br>覆盖后续所有配置]
    B -- 未配置 --> D{检查方式2<br>ASPNETCORE_URLS环境变量}
    D -- 已设置 --> E[使用此环境变量<br>覆盖后续配置]
    D -- 未设置 --> F{检查方式3<br>命令行 --urls 参数}
    F -- 已提供 --> G[使用命令行参数]
    F -- 未提供 --> H{检查方式4<br>代码中的 UseUrls}
    H -- 已调用 --> I[使用代码中指定的地址]
    H -- 未调用 --> J[最终方式5<br>回退至 launchSettings.json]
    J --> K[使用 applicationUrl 配置]
```


> [!caution] 服务托管到 IIS 时配置逻辑完全不同
> 在 IIS 托管下，`applicationUrl` 和 `Kestrel:Endpoints` 的设置通常会被完全忽略，实际的 URL 和端口由 IIS 站点绑定决定。
> 
> 这是由于在 IIS 托管模式下，.NET Core 应用（Kestrel 服务器）并不直接对外服务，而是作为一个后台进程运行，只与 IIS 工作进程（`w3wp.exe`）通信。IIS 充当了一个反向代理，接收外部请求并转发给内部的 Kestrel 进程，这个内部转发地址通常由 ASP.NET Core 模块（负责桥接 IIS 和 Kestrel 的模块）**自动处理**。

