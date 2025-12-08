
## 部署方案

### 部署到 Docker

**`.gitlab-ci.yml`**：
```yaml
# GitLab CI/CD

stages:
  - build
  - test
  - deploy

variables:
  SOLUTION: "ThBIM.ASPNETCore.WebService.sln"
  BUILD_CONFIGURATION: "Release"
  DOTNET_VERSION: "8.0.x"
  PUBLISH_OUTPUT_DIR: "publish"

  # 部署相关变量
  DOCKER_IMAGE: "your-registry/your-app"
  DOCKER_TAG: "$CI_COMMIT_SHORT_SHA"

cache:
  key: "$CI_COMMIT_REF_SLUG"
  paths:
    - .nuget/packages
  policy: pull-push

workflow:
  rules:
    - if: $CI_COMMIT_BRANCH == "ThBIM.ASPNETCore"

# 添加 Docker 构建作业
build-docker-image:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t $DOCKER_IMAGE:$DOCKER_TAG -t $DOCKER_IMAGE:latest .
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker push $DOCKER_IMAGE:$DOCKER_TAG
    - docker push $DOCKER_IMAGE:latest
  needs: ["build-solution"]
  rules:
    - if: $CI_COMMIT_BRANCH == "ThBIM.ASPNETCore"

# 添加 Docker 部署作业
deploy-docker:
  stage: deploy
  image: alpine/ssh
  script:
    - |
      ssh -o StrictHostKeyChecking=no $DEPLOY_USER@$DEPLOY_SERVER "
        # 拉取最新镜像
        docker pull $DOCKER_IMAGE:latest
        
        # 停止并移除旧容器
        docker stop your-app-container || true
        docker rm your-app-container || true
        
        # 运行新容器
        docker run -d \
          --name your-app-container \
          -p 8080:80 \
          -e ASPNETCORE_ENVIRONMENT=Production \
          $DOCKER_IMAGE:latest
      "
  needs: ["build-docker-image"]
  rules:
    - if: $CI_COMMIT_BRANCH == "ThBIM.ASPNETCore"
      when: manual
```


### 部署到 IIS（Windows Server）

**`.gitlab-ci.yml`**：
```yaml
# GitLab CI/CD

stages:
  - build
  - test
  - deploy

variables:
  SOLUTION: "ThBIM.ASPNETCore.WebService.sln"
  BUILD_CONFIGURATION: "Release"
  DOTNET_VERSION: "8.0.x"
  PUBLISH_OUTPUT_DIR: "publish"

  # 部署相关变量
  DEPLOY_SERVER: "your-server-ip"
  DEPLOY_USER: "administrator"
  DEPLOY_PATH: "C:\WebSites\YourApp"
  IIS_SITE_NAME: "YourAppSite"
  IIS_APP_POOL: "YourAppPool"

cache:
  key: "$CI_COMMIT_REF_SLUG"
  paths:
    - .nuget/packages
  policy: pull-push

workflow:
  rules:
    - if: $CI_COMMIT_BRANCH == "ThBIM.ASPNETCore"


deploy-to-iis:
  stage: deploy
  tags:
    - win64-aspnetcore-runner  # 确保这个 runner 可以访问目标服务器
  script:
    - Write-Host "开始部署到 IIS..." -ForegroundColor Green
    
    # 方式一：使用 WinRM（推荐）
    - |
      $securePassword = ConvertTo-SecureString "$DEPLOY_PASSWORD" -AsPlainText -Force
      $credential = New-Object System.Management.Automation.PSCredential("$DEPLOY_USER", $securePassword)
      
      # 复制文件到服务器
      $session = New-PSSession -ComputerName "$DEPLOY_SERVER" -Credential $credential
      
      # 停止应用池
      Invoke-Command -Session $session -ScriptBlock {
        Stop-WebAppPool -Name "$env:IIS_APP_POOL"
      }
      
      # 删除旧文件（保留备份可选）
      Invoke-Command -Session $session -ScriptBlock {
        if (Test-Path "$env:DEPLOY_PATH") {
          Remove-Item -Path "$env:DEPLOY_PATH\*" -Recurse -Force
        }
      }
      
      # 复制新文件
      Copy-Item -Path "$CI_PROJECT_DIR\$PUBLISH_OUTPUT_DIR\*" -Destination "\\$env:DEPLOY_SERVER\$env:DEPLOY_PATH" -Recurse -Force
      
      # 启动应用池
      Invoke-Command -Session $session -ScriptBlock {
        Start-WebAppPool -Name "$env:IIS_APP_POOL"
        Start-Website -Name "$env:IIS_SITE_NAME"
      }
      
      Remove-PSSession -Session $session
      
    Write-Host "部署完成！" -ForegroundColor Green
    
    # 方式二：简单的文件复制（如果 runner 和目标服务器在同一网络）
    # Copy-Item -Path "$CI_PROJECT_DIR\$PUBLISH_OUTPUT_DIR\*" -Destination "\\$DEPLOY_SERVER\c$\WebSites\YourApp\" -Recurse -Force
    
    # 重启 IIS（如果需要）
    # Invoke-Command -ComputerName $DEPLOY_SERVER -ScriptBlock { iisreset }
    
  needs: ["build-solution"]
  dependencies:
    - build-solution
  rules:
    - if: $CI_COMMIT_BRANCH == "ThBIM.ASPNETCore"
      when: manual  # 设置为手动触发部署
    - when: never
```


### FTP 部署

**`.gitlab-ci.yml`**：
```yaml
# GitLab CI/CD

stages:
  - build
  - test
  - deploy

variables:
  SOLUTION: "ThBIM.ASPNETCore.WebService.sln"
  BUILD_CONFIGURATION: "Release"
  DOTNET_VERSION: "8.0.x"
  PUBLISH_OUTPUT_DIR: "publish"

  # 部署相关变量
  DEPLOY_SERVER: "your-server-ip"
  DEPLOY_USER: "administrator"
  DEPLOY_PATH: "C:\WebSites\YourApp"
  IIS_SITE_NAME: "YourAppSite"
  IIS_APP_POOL: "YourAppPool"

cache:
  key: "$CI_COMMIT_REF_SLUG"
  paths:
    - .nuget/packages
  policy: pull-push

workflow:
  rules:
    - if: $CI_COMMIT_BRANCH == "ThBIM.ASPNETCore"


deploy-ftp:
  stage: deploy
  tags:
    - win64-aspnetcore-runner
  script:
    - Write-Host "通过 FTP 部署..." -ForegroundColor Green
    
    # 安装 WebClient（如果未安装）
    - Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force
    - Install-Module -Name WebClient -Force -AllowClobber
    
    # 使用 FTP 上传
    - |
      $ftpServer = "ftp://$FTP_SERVER"
      $ftpUser = "$FTP_USERNAME"
      $ftpPass = "$FTP_PASSWORD"
      
      $webclient = New-Object System.Net.WebClient
      $webclient.Credentials = New-Object System.Net.NetworkCredential($ftpUser, $ftpPass)
      
      # 上传所有文件
      $files = Get-ChildItem -Path "$CI_PROJECT_DIR\$PUBLISH_OUTPUT_DIR" -Recurse
      
      foreach ($file in $files) {
        $relativePath = $file.FullName.Substring($CI_PROJECT_DIR.Length + $PUBLISH_OUTPUT_DIR.Length + 1)
        $uri = New-Object System.Uri("$ftpServer/$relativePath")
        
        if ($file.PSIsContainer) {
          # 创建目录
          try {
            $makeDirectory = [System.Net.WebRequest]::Create($uri)
            $makeDirectory.Credentials = New-Object System.Net.NetworkCredential($ftpUser, $ftpPass)
            $makeDirectory.Method = [System.Net.WebRequestMethods+Ftp]::MakeDirectory
            $makeDirectory.GetResponse()
          } catch {}
        } else {
          # 上传文件
          $webclient.UploadFile($uri, $file.FullName)
        }
      }
  needs: ["build-solution"]
  rules:
    - if: $CI_COMMIT_BRANCH == "ThBIM.ASPNETCore"
      when: manual
```


## 环境变量设置

在 GitLab 项目的 **Settings > CI/CD > Variables** 中添加以下变量：

1. **对于 IIS 部署**：
    - `DEPLOY_PASSWORD` - 服务器密码（设置为 masked）
    - `DEPLOY_SERVER` - 服务器 IP/主机名
    - `DEPLOY_USER` - 用户名
2. **对于 Docker 部署**：
    - `CI_REGISTRY_USER` - Docker registry 用户名
    - `CI_REGISTRY_PASSWORD` - Docker registry 密码（设置为 masked）
3. **对于 FTP 部署**：
    - `FTP_SERVER` - FTP 服务器地址
    - `FTP_USERNAME` - FTP 用户名
    - `FTP_PASSWORD` - FTP 密码（设置为 masked）

