在 Windows 上，可以使用微软提供的一整套自动化解决方案。

## 核心配置

### **GitLab Runner 注册（Windows）**

```powershell
# 1. 下载 Windows binary


# 2. 以管理员运行注册
gitlab-runner register `
  --url "https://gitlab.com/" `
  --registration-token "PROJECT_TOKEN" `
  --executor "shell" `
  --shell "powershell" `
  --description "windows-runner"

# 3. 安装为服务
gitlab-runner install
gitlab-runner start
```


### GitLab CI 配置

1. **`.gitlab-ci.yml` 配置示例**
```yaml
stages:
  - build
  - test
  - deploy

variables:
  DOTNET_VERSION: "8.0.x"
  PROJECT_PATH: "src/WebApp/WebApp.csproj"
  PUBLISH_OUTPUT: "publish"
  IIS_SITE_NAME: "MyApp"
  DEPLOY_SERVER: "win-server.prod"

# 缓存 NuGet 包
cache:
  paths:
    - .nuget/

before_script:
  - chcp 65001  # 设置 UTF-8 编码
  - dotnet --version

build_job:
  stage: build
  script:
    - dotnet restore $PROJECT_PATH
    - dotnet build $PROJECT_PATH --configuration Release --no-restore
    - dotnet publish $PROJECT_PATH --configuration Release --output $PUBLISH_OUTPUT --runtime win-x64
  artifacts:
    paths:
      - $PUBLISH_OUTPUT/
    expire_in: 1 week

test_job:
  stage: test
  script:
    - dotnet test --configuration Release --no-build --verbosity normal
  dependencies:
    - build_job

deploy_prod:
  stage: deploy
  script:
    - powershell -File .\deploy.ps1 -Server $DEPLOY_SERVER -SiteName $IIS_SITE_NAME
  environment:
    name: production
    url: https://app.prod.com
  only:
    - main
```

2. **部署脚本 `deploy.ps1`**
```powershell
param(
    [string]$Server,
    [string]$SiteName,
    [string]$SourcePath = "publish",
    [string]$AppPoolName = $SiteName
)

# 1. 通过 WinRM 连接服务器
$sessionOptions = New-PSSessionOption -SkipCACheck -SkipCNCheck
$session = New-PSSession -ComputerName $Server -SessionOption $sessionOptions -Credential (Get-Credential)

# 2. 远程执行部署
Invoke-Command -Session $session -ScriptBlock {
    param($Site, $Pool, $Path)
    
    # 停止应用池
    Stop-WebAppPool -Name $Pool -ErrorAction SilentlyContinue
    
    # 备份现有版本
    $backupPath = "D:\Backups\$Site\$(Get-Date -Format 'yyyyMMdd_HHmm')"
    if (Test-Path "D:\Websites\$Site") {
        Copy-Item "D:\Websites\$Site" $backupPath -Recurse -Force
    }
    
    # 清空目标目录
    Remove-Item "D:\Websites\$Site\*" -Recurse -Force -ErrorAction SilentlyContinue
    
    # 复制新版本
    Copy-Item "$Path\*" "D:\Websites\$Site\" -Recurse -Force
    
    # IIS 配置
    if (-not (Get-WebAppPool -Name $Pool -ErrorAction SilentlyContinue)) {
        New-WebAppPool -Name $Pool -Force
        Set-ItemProperty "IIS:\AppPools\$Pool" -Name managedRuntimeVersion -Value ""
        Set-ItemProperty "IIS:\AppPools\$Pool" -Name startMode -Value "AlwaysRunning"
        Set-ItemProperty "IIS:\AppPools\$Pool" -Name recycling.periodicRestart.time -Value "00:00:00"
    }
    
    if (-not (Get-Website -Name $Site -ErrorAction SilentlyContinue)) {
        New-Website -Name $Site -PhysicalPath "D:\Websites\$Site" -ApplicationPool $Pool -Port 80 -Force
    }
    
    # 启动应用池
    Start-WebAppPool -Name $Pool
    Start-Website -Name $Site
    
    # 健康检查
    Start-Sleep -Seconds 10
    $status = (Get-Website -Name $Site).State
    Write-Host "网站状态: $status"
    
} -ArgumentList $SiteName, $AppPoolName, $SourcePath

Remove-PSSession $session
```


### 服务器端配置

1. **配置**
```powershell
# 1. 启用 WinRM
Enable-PSRemoting -Force

# 2. 配置信任的主机（或使用 HTTPS）
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "gitlab-runner-ip" -Force

# 3. 修改基础认证（生产环境建议用 HTTPS + 证书）
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "LocalAccountTokenFilterPolicy" -Value 1

# 4. 重启服务
Restart-Service WinRM
```

2. **IIS 优化配置（PowerShell）**
```powershell
# 1. 安装 IIS 功能
Install-WindowsFeature Web-WebSockets, Web-Asp-Net45, Web-Windows-Auth

# 2. 应用池配置优化
Set-ItemProperty "IIS:\AppPools\$AppPoolName" -Name processModel.idleTimeout -Value "00:00:00"
Set-ItemProperty "IIS:\AppPools\$AppPoolName" -Name processModel.pingingEnabled -Value $true
Set-ItemProperty "IIS:\AppPools\$AppPoolName" -Name recycling.periodicRestart.time -Value "00:00:00"
```

### 进阶配置技巧

1. **使用安全凭据**
```yaml
# 在 GitLab CI/CD 变量中设置（掩码保护）
# DEPLOY_USER / DEPLOY_PASSWORD
script:
  - $securePass = ConvertTo-SecureString $env:DEPLOY_PASSWORD -AsPlainText -Force
  - $credential = New-Object System.Management.Automation.PSCredential ($env:DEPLOY_USER, $securePass)
  - $session = New-PSSession -ComputerName $Server -Credential $credential
```

2. **回滚机制**
```powershell
# 在 deploy.ps1 中添加回滚函数
function Rollback-Deployment {
    param($Site, $BackupPath)
    if (Test-Path $BackupPath) {
        Stop-Website -Name $Site
        Remove-Item "D:\Websites\$Site\*" -Recurse -Force
        Copy-Item "$BackupPath\*" "D:\Websites\$Site\" -Recurse -Force
        Start-Website -Name $Site
    }
}
```


### 容器化替代方案（Windows Containers）

```yaml
# 使用 Docker 执行器
windows_runner:
  tags:
    - windows
    - docker
  image: mcr.microsoft.com/dotnet/sdk:8.0-nanoserver-ltsc2022
  script:
    - dotnet publish -c Release -o output
```


### 监控与日志

```powershell
# 1. 部署后验证
Invoke-WebRequest -Uri "https://app.prod.com/health" -UseBasicParsing

# 2. 集成 Application Insights
# 在 appsettings.json 中添加
{
  "ApplicationInsights": {
    "ConnectionString": "InstrumentationKey=..."
  }
}
```

