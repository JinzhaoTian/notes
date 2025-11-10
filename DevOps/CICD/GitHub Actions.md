GitHub Actions 是 [GitHub](../Code%20Version/GitHub.md) 原生支持的持续集成（CI）和持续交付/部署（CD）平台，允许开发者直接在 GitHub 仓库中自动化构建、测试和部署流程，无需额外配置服务器。

- **公共仓库**：GitHub Actions 对于 GitHub-hosted runners 和 self-hosted runners 均免费使用。
- **私有仓库**：每个 GitHub 帐户都会获得一定数量的免费分钟数和存储空间，可用于 GitHub-hosted runners。


## 核心概念

1. **Workflow（工作流）**
    - 一个可配置的自动化流程，存储在 `.github/workflows/` 目录下的 YAML 文件中。
    - 由事件（Events）触发（如 `push`、`pull_request`、`schedule` 等）。
2. **Job（任务）**
    - 一个工作流包含多个任务，每个任务由一系列步骤（Steps）组成。
    - 任务可以并行或顺序执行，运行在 GitHub 托管的虚拟机（如 `ubuntu-latest`）或自托管服务器上。
3. **Step（步骤）**
    - 单个任务的具体操作，可以是：
        - 运行命令（如 `npm install`）。
        - 使用预定义的 Action
4. **Action（动作）**
    - 可复用的代码单元，用于简化流程（如检出代码、设置 Node.js 环境）。
    - 可以是 GitHub 官方提供的（如 `actions/checkout`），也可以是社区开发的。


## 核心机制

- **自动化测试**：代码提交后自动运行测试。
- **自动部署**：推送代码到 `main` 分支时部署到服务器或云平台（如 AWS、Vercel）。
- **计划任务**：定时运行任务（如每日构建）。
- **多平台支持**：支持 Linux、Windows、macOS 等环境。
- **矩阵构建**：同时测试多个版本（如不同 Node.js 或 Python 版本）。
- **Artifacts**：存储构建产物（如日志、二进制文件）。


### 事件驱动的工作流

GitHub Actions 的工作流由 `.github/workflows/*.yml` 文件定义，可以在特定事件（如 `push`、`pull_request`、`schedule` 等）触发时自动运行。

```yaml
on:
    push:
        branches: [ "main" ]
    pull_request:
        branches: [ "main" ]
```


### 多环境支持

GitHub Actions 提供多种运行环境（`runs-on`），包括：
- **GitHub-hosted runners**：GitHub 官方提供的不同操作系统的镜像，[actions/runner-images: GitHub Actions runner images](https://github.com/actions/runner-images)
	- **Ubuntu**
	- **Windows**
	- **macOS**
- **self-hosted runners**：用户可以在自己的服务器或云实例上运行任务，适用于特殊硬件需求或私有环境。

```yaml
jobs:
    test:
        runs-on: ubuntu-latest
```

### 矩阵构建

矩阵构建（Matrix Builds）支持并行测试多个环境组合（如不同 Node.js 版本、操作系统等），减少测试时间

```yaml
strategy:
    matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: [14.x, 16.x]
```


### Actions 复用

GitHub [Marketplace](https://github.com/marketplace?type=actions) 提供大量现成的 Actions（如 `actions/checkout@v4`、`actions/setup-node@v4`），可直接引用


### 敏感数据管理

通过 GitHub Secrets 安全存储敏感信息（如 API 密钥、服务器凭据），避免硬编码在代码中

```yaml
env:
    API_KEY: ${{ secrets.MY_API_KEY }}
```


### 部署自动化

支持多种部署方式，如：
- **SSH/SCP 上传**：将构建产物推送到服务器6。
- **云平台集成**：部署到 AWS、Heroku、Vercel 等5。
- **自定义脚本**：通过 `run` 执行部署命令。

```yaml
- name: Deploy via SSH
  uses: appleboy/scp-action@master
  with:
    host: ${{ secrets.SSH_HOST }}
    username: ${{ secrets.SSH_USER }}
    key: ${{ secrets.SSH_PRIVATE_KEY }}
    source: "dist/*"
    target: "/var/www/html"
```

### 条件执行与依赖控制

- **条件执行（if）**：根据分支、事件类型等决定是否运行任务
- **依赖关系（needs）**：确保任务按顺序执行（如先构建后部署）

```yaml
jobs:
    build:
        runs-on: ubuntu-latest
    deploy:
        needs: build
        if: github.ref == 'refs/heads/main'
```


### 日志与监控

- **实时日志**：在 GitHub Actions 控制台查看任务执行状态
- **Codecov 集成**：上传测试覆盖率报告
- **Artifacts**：存储构建产物供后续使用






## 优势

1. **深度集成 GitHub**：无需第三方工具即可实现 CI/CD。
2. **灵活的配置**：YAML 文件定义流程，易于版本控制。
3. **丰富的 Action 市场**：可直接使用数千种社区 Action。
4. **免费额度**：公开仓库和私有仓库（有限制）均可免费使用。


## 使用步骤

在项目根目录创建 `.github/workflows` 文件夹，

