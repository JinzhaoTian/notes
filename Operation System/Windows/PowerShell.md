
## 命令

#### `New-Object`

用于创建 .NET Framework 或 [COM](COM.md) 对象实例的 `cmdlet`，允许开发者通过指定类型名称或 COM 对象的 ProgID 来实例化对象。
```powershell
New-Object
    [-TypeName] <String>
    [[-ArgumentList] <Object[]>]
    [-Property <IDictionary>]
    [<CommonParameters>]
```

#### `Copy-Item`

用于将 Item 从一个位置复制到另一个位置。
```powershell
Copy-Item
    [-Path] <String[]>
    [[-Destination] <String>]
    [-Container]
    [-Force]
    [-Filter <String>]
    [-Include <String[]>]
    [-Exclude <String[]>]
    [-Recurse]
    [-PassThru]
    [-Credential <PSCredential>]
    [-WhatIf]
    [-Confirm]
    [-FromSession <PSSession>]
    [-ToSession <PSSession>]
    [<CommonParameters>]
```

**参数**：
- **`Path`**：源文件路径（通常是构建输出目录）
- **`Destination`**：目标部署路径
- **`Recurse`**：递归复制子目录
- **`Force`**：强制覆盖现有文件
- **`ToSession`**：复制到远程会话（通过 PSSession）

#### `Invoke-Command`

用于在本地或远程计算机上运行命令，并返回所有输出，包括错误。

```powershell
Invoke-Command 
	[-ScriptBlock] <ScriptBlock> 
	[[-ComputerName] <String[]>]
	[-Credential <PSCredential>]
	[-ArgumentList <Object[]>]
	[<CommonParameters>]
```

**参数**：
- **`ComputerName`**：指定运行命令的远程计算机。
- **`Credential`**：指定有权执行操作的用户帐户。
- **`ScriptBlock`**：指定要运行的命令。
- **`ArgumentList`**：提供脚本块的参数值。


#### `New-PSSession`

用于创建本地或远程计算机 PowerShell 会话（PSSession）的命令。通过 PSSession，用户可以建立与目标计算机的持久连接，从而在远程环境中执行命令或脚本。

```powershell
New-PSSession
    [[-ComputerName] <String[]>]
    [-Credential <PSCredential>]
    [-Name <String[]>]
    [-EnableNetworkAccess]
    [-ConfigurationName <String>]
    [-Port <Int32>]
    [-UseSSL]
    [-ApplicationName <String>]
    [-ThrottleLimit <Int32>]
    [-SessionOption <PSSessionOption>]
    [-Authentication <AuthenticationMechanism>]
    [-CertificateThumbprint <String>]
    [<CommonParameters>]
```



#### `Remove-PSSession`

关闭一个或多个 PowerShell 会话（PSSessions）。

```powershell
Remove-PSSession
    [-Id] <Int32[]>
    [-WhatIf]
    [-Confirm]
    [<CommonParameters>]
```
