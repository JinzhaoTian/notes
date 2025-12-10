GitLab Runner 是 GitLab CI/CD 中用于执行任务的应用程序，是 GitLab CI/CD 流水线的核心组件，负责在不同环境中运行在 `.gitlab-ci.yml` 文件中定义的构建、测试、部署等任务，没有 Runner，GitLab 无法执行任何任务。

## 使用

1. **安装 GitLab Runner**：可以在各种操作系统上安装 GitLab Runner，包括 Linux、macOS 和 Windows，参考[文档](https://docs.gitlab.com/runner/install/)。
2. **注册 GitLab Runner**：安装完 Runner 后，需要在 Runner 所在的服务器上运行注册命令，并提供 GitLab 实例的 URL 和项目或组的注册令牌。执行 `gitlab-runner register` 命令时，会让你选择执行器并设置标签。
3. **使用标签**：当你在 `.gitlab-ci.yml` 文件中定义任务时，可以使用 `tags` 字段指定使用哪些 Runner。每个 Runner 注册时可以定义一些标签，GitLab 会根据标签匹配合适的 Runner 来执行任务。

## 安装

#### Windows

1. **创建**一个本地目录，如 `C:\GitLab-Runner`
2. **下载**二进制包（[64-bit](https://s3.dualstack.us-east-1.amazonaws.com/gitlab-runner-downloads/latest/binaries/gitlab-runner-windows-amd64.exe) 或者 [32-bit](https://s3.dualstack.us-east-1.amazonaws.com/gitlab-runner-downloads/latest/binaries/gitlab-runner-windows-386.exe)）到这个本地目录，假设已经将这个 exe 重命名为 `gitlab-runner.exe`
	- **确保本地目录与 `gitlab-runner.exe` 有写权限**
3. **注册** GitLab Runner：
```bash
cd C:\GitLab-Runner
.\gitlab-runner.exe register
```
- 输入：
	- 输入 GitLab instance URL，如果项目地址是 `gitlab.example.com/yourname/yourproject`，此时应该输入 `https://gitlab.example.com`
	- 输入 GitLab Runner 的 authentication token
	- 输入 GitLab Runner 的描述
	- 输入 job tags，以逗号 `,` 隔开
	- 输入 GitLab Runner 的 optional maintenance note
	- 输入 GitLab Runner Executor（执行器）的类型
		- 常见的有：shell、ssh、Docker
4. 将 GitLab Runner 作为一个服务进行**安装**并**启动**
```bash
cd C:\GitLab-Runner
.\gitlab-runner.exe install
.\gitlab-runner.exe start
```

其中，在 GitLab 里可以直接创建 Runner：
![](_imgs/Pasted%20image%2020251205163527.png)
![](_imgs/Pasted%20image%2020251205163548.png)
![](_imgs/Pasted%20image%2020251205163735.png)

按照说明在目标机器上运行即可。


## 执行器

GitLab Runner 的执行器 Executor 是指 Runner 执行 CI/CD 作业时使用的执行环境类型，决定了作业在什么样的环境中运行、如何隔离、以及资源如何分配。

> [!tip] 无论使用哪种执行器，第一步都是获取项目代码
> GitLab CI/CD 作业的标准流程，无论使用哪种执行器，第一步都是获取项目代码。

#### Shell

指定 Executor 为 Shell 类型，将直接在 Runner 所在机器的系统 Shell 中运行作业。

**特点**：
1. 最简单，无隔离，直接使用主机环境。
2. 依赖和缓存都在主机上。
3. 安全性低（作业可以任意访问主机）。

**常见类型**：
1. Bash
2. PowerShell Core
3. Windows PowerShell

**默认配置**：
1. **获取项目代码**：当作业启动时，Runner 会自动将仓库克隆到本地一个特定的目录中，根据官方文档，其默认路径结构如下：
```
<working-directory>/builds/<short-token>/<concurrent-id>/<namespace>/<project-name>
```
- **含义**：
	- `<working-directory>`：启动 `gitlab-runner` 时指定的 `--working-directory`，或 Runner 的当前目录。
	- `<short-token>`：Runner 注册令牌的前 8 个字符。
	- `<concurrent-id>`：并发ID，用于唯一标识该 Runner 上某个作业的 ID。
	- `<namespace>`：命名空间
	- `<project-name>`：项目名称，对应在 GitLab 上的项目路径。


#### Docker

指定 Executor 为 Docker 类型，将在每个作业中启动一个 Docker 容器来运行任务。

**特点**：
1. 环境隔离性好，依赖隔离。
2. 通过 `image` 指定容器镜像。
3. 支持服务容器（如 MySQL、Redis）。

**配置示例**（`.gitlab-ci.yml`）：
```yaml
job1:
  image: alpine:latest
  script:
    - echo "运行在 Docker 容器中"
```

#### Docker Machine

指定 Executor 为 Docker Machine 类型，类似 Docker，但会自动创建和管理 Docker 主机（适用于动态伸缩场景）。

**特点**：
1. 适用于云环境，自动创建虚拟机来运行 Docker 容器。
2. 弹性伸缩，闲置时自动释放资源。


#### Kubernetes

指定 Executor 为 Kubernetes 类型，将在 Kubernetes 集群中创建 Pod 来运行作业。

**特点**：
1. 强大的资源调度和隔离。
2. 原生集成 K8s 生态（Secrets、ConfigMap 等）。

#### VirtualBox/Parallels

指定 Executor 为 VirtualBox/Parallels 类型，将通过虚拟机运行作业（使用 Vagrant 管理）。

**特点**：
1. 完全隔离的操作系统级环境。
2. 性能开销较大。

#### SSH

指定 Executor 为 VirtualBox/Parallels 类型，将通过 SSH 连接到远程服务器执行命令。

**特点**：
1. 在指定远程主机上运行。
2. 需提前配置好环境。

#### Custom

指定 Executor 为 VirtualBox/Parallels 类型，将用户自定义执行方式（需自己实现执行逻辑）。

**特点**：
1. 高度灵活，可对接任意系统。


