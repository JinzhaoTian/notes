IIS（Internet Information Services，互联网信息服务）是微软公司开发的 Web 服务器软件，运行在 Windows 操作系统上，用于托管网站、Web 应用程序和 Web 服务。

## 主要特点

1. **集成于 Windows 系统**
    - IIS 是 Windows Server 的核心组件（也包含在 Windows 专业版/企业版中），与 Windows 生态系统（如 Active Directory、.NET Framework）深度集成。
2. **支持多种 Web 技术**
    - **ASP.NET**：微软的 Web 开发框架，与 IIS 高度协同。
    - **PHP**、**Python**等：通过扩展模块支持。
    - **静态文件**（HTML、CSS、JS）和动态内容处理。
3. **管理工具**
    - **IIS 管理器**：图形化界面，便于配置网站、虚拟目录、SSL 证书等。
    - **PowerShell 命令**：支持自动化管理。
4. **安全性功能**
    - 集成 Windows 身份验证（如 Kerberos）。
    - 支持 HTTPS/SSL、请求过滤、IP 限制等。
5. **可扩展性**
    - 可通过模块（如 URL 重写、负载均衡）增强功能。
    - 与 Azure 云服务无缝集成。


## 应用场景

1. **企业内网网站**：托管公司内部门户或应用。
2. **公共网站**：运行 ASP.NET 等微软技术栈的网站。
3. **Web API 服务**：作为后端服务的宿主。
4. **文件传输**：支持 FTP 服务（通过 IIS 的 FTP 模块）。


### 托管 Web API 服务

以 .NET 8 的 ASP.NET Core 服务托管为例，核心是配置 IIS 作为反向代理，将请求转发给应用程序内部的 Kestrel 服务器。

**托管步骤**：

1. **环境准备**
	- 安装 IIS 与 .NET Hosting Bundle，如确保服务器安装 .NET 8 Runtime 或 .NET 8 Hosting Bundle。
2. **发布应用程序**
	- 使用 `dotnet publish` 发布，发布方式选择依赖框架或独立部署会直接影响 `web.config` 的配置
3. **配置 IIS 站点**
	- 创建网站，指向发布文件夹，应用程序池需设为 “无托管代码” 和 “集成管道模式”
4. **配置 `web.config`**
	- 设置处理程序与模块，核心配置是指定由 **`AspNetCoreModuleV2`** 处理请求
5. **解决文件锁定**：避免 IIS 进程锁定应用文件

#### 核心原理

IIS 通过 ASP.NET Core 模块（ANCM）与你的应用程序通信，支持两种运行方式：
1. **进程内托管**：应用直接在 IIS 工作进程（`w3wp.exe`）中运行，性能更好，是 .NET Core 3.0 及更高版本的**默认方式**
2. **进程外托管**：IIS 作为反向代理，将请求转发给后端独立的 Kestrel 服务器进程

对于 .NET 8，通常推荐使用默认的**进程内托管**以获得最佳性能。


#### 关键配置


