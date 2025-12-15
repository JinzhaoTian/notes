Windows Containers 是微软在 Windows Server 和 Windows 10/11 上提供的容器化技术，允许将应用程序及其依赖项打包为独立的、可移植的容器单元。

Windows Containers 特别适合需要将现有 Windows 应用程序现代化，或必须在 Windows 环境下运行的应用场景。虽然生态系统不如 Linux 容器成熟，但在微软技术栈中具有重要地位。

## 核心特性

![](../../DevOps/Container/_imgs/Pasted%20image%2020251215101547.png)
![](../../DevOps/Container/_imgs/Pasted%20image%2020251215101630.png)

1. **两种容器类型**
	- **Windows Server Containers**：进程隔离，共享主机内核
	- **Hyper-V Containers**：通过轻量级虚拟机提供更强的隔离性

2. **基础镜像类型**
	- **Nano Server**：极简镜像（~250MB），适用于 .NET Core 和 PowerShell 应用
	- **Server Core**：中等大小（~1.5GB），支持传统 .NET Framework 应用
	- **Windows**：完整桌面体验镜像（~4.5GB），包含 GUI 支持

> [!tip] 与 Linux 容器的关键差异
> |方面|Windows Containers|Linux Containers|
|---|---|---|
|**内核**|Windows NT 内核|Linux 内核|
|**文件系统**|NTFS|ext4/xfs等|
|**网络栈**|Windows网络栈|Linux网络栈|
|**镜像格式**|基于层级VHD|基于层级文件系统|

## 使用场景

1. 现代化遗留 .NET Framework 应用
2. IIS 托管的 Web 应用
3. SQL Server 容器化
4. 需要在 Windows 特定 API 的应用程序
5. 混合环境中的 Windows 服务

> [!tip] 限制
> 1. 只能在 Windows 主机上运行
> 2. 镜像通常比 Linux 容器大
> 3. 生态系统相对较小
> 4. 某些 Linux 容器功能可能不支持


## 技术栈

```
├── 容器运行时
│   ├── Docker Engine for Windows
│   ├── containerd
│   └── CRI-O (有限支持)
├── 编排平台
│   ├── Kubernetes (AKS, AKS on HCI)
│   ├── Service Fabric
│   └── Docker Swarm
└── 开发工具
    ├── Visual Studio / VS Code
    ├── Docker Desktop for Windows
    └── Windows Terminal
```


## 使用示例

```dockerfile
# Dockerfile示例 - Windows容器
FROM mcr.microsoft.com/dotnet/framework/aspnet:4.8-windowsservercore-ltsc2019

# 设置工作目录
WORKDIR /inetpub/wwwroot

# 复制应用文件
COPY MyWebApp/ .

# 暴露端口
EXPOSE 80
```

## 部署

1. **启用**
```powershell
# 启用Windows容器功能
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V, Containers -All
```

2. **基础镜像**
```powershell
# 拉取基础镜像
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

3. **云端部署**
	- **Azure Container Instances（ACI）**
	- **Azure Kubernetes Service（AKS）**
	- **Azure App Service（Windows容器支持）**

4. **混合云**
	- Azure Arc enabled Kubernetes
	- Windows Server 2019/2022 with Kubernetes