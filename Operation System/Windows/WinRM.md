WinRM（Windows Remote Management，Windows 远程管理）是微软基于 WS-Management 协议实现的一项功能，允许管理员通过网络对本地或远程的 Windows 主机进行安全地管理。

> [!caution]
> WinRM 是 Windows 自动化运维、DevOps（如 Ansible 管理 Windows 节点）以及云环境（如 Azure VM 扩展）中的关键组件。


## 主要功能

1. **远程命令执行**
    - 通过命令行（如 PowerShell）或 API 远程运行命令或脚本。
    - 例如，使用 PowerShell 的 `Invoke-Command` 连接 WinRM 服务执行远程命令。
2. **系统配置管理**
    - 用于配置远程计算机（如修改注册表、服务、事件日志等）。
    - 是 PowerShell Desired State Configuration（DSC）的底层传输协议之一。
3. **硬件与软件清单收集**
    - 可远程查询系统信息、安装的软件、硬件配置等。
4. **跨平台管理**
    - WinRM 虽然主要针对 Windows，但也支持通过 OpenWSMAN 等工具与 Linux 系统通信。

## 技术特点

1. **协议**：基于 SOAP over HTTP/HTTPS（默认端口 HTTP 5985 / HTTPS 5986）。
2. **认证方式**：
    - 本地账户（NTLM/Kerberos）
    - 域账户（Kerberos）
    - 基于证书的认证
    - 基本认证（需配合 HTTPS）
3. **默认配置**：Windows Server 2008 及更高版本内置，但默认通常**未启用**（Windows Server 2012 及以后部分版本自动启用）。

## 启用与配置

1. **使用 PowerShell（管理员权限）**
```powershell
# 启用 WinRM 服务
Enable-PSRemoting -Force
```

2. **查看 WinRM 配置**
```powershell
winrm get winrm/config
```

3. **允许远程连接（非域环境时可能需要）**
```powershell
winrm set winrm/config/client '@{TrustedHosts="192.168.1.*"}'
```


## 使用示例

1. **通过 PowerShell 远程执行命令**
```powershell
# 在远程计算机上运行命令
Invoke-Command -ComputerName "Server01" -ScriptBlock { Get-Process } -Credential (Get-Credential)
```

2. **建立远程 PowerShell 会话**
```powershell
$session = New-PSSession -ComputerName "Server01" -Credential (Get-Credential)
Enter-PSSession $session
```