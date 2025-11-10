Jenkins 是一个开源的持续集成（CI）和持续交付/部署（CD）工具，用于自动化软件开发过程中的构建、测试、部署等任务。


## 核心功能

1. **持续集成（CI）**
    - 自动触发代码构建和测试（如 Git 提交后）。
    - 快速发现代码集成错误。
2. **持续交付/部署（CD）**
    - 自动化部署到测试、预发布或生产环境。
    - 支持滚动发布、蓝绿部署等策略。
3. **任务自动化**
    - 执行脚本（Shell、Python 等）。
    - 定时任务（如 nightly builds）。
4. **插件生态系统**
    - 提供 1500+ 插件，支持与 Git、Docker、Kubernetes、AWS、JIRA 等工具集成。
5. **分布式构建**
    - 在多台机器上并行执行任务，加速构建过程。


## 工作原理

1. **代码变更**：开发者提交代码到版本控制系统（如 GitHub/GitLab）。
2. **触发构建**：Jenkins 检测到变更后，自动启动构建任务（通过 Webhook 或轮询）。
3. **构建与测试**：执行编译、单元测试、代码检查等。
4. **反馈结果**：生成测试报告，通知开发人员（如通过邮件/Slack）。
5. **部署**：若构建成功，自动部署到指定环境。


## 应用场景

- 自动构建 Java/Go/.NET/Node 项目。
- 部署 Docker 容器到 Kubernetes。
- 执行自动化测试（Selenium、JMeter）。
- 与 SonarQube 结合实现代码质量检查。


## 安装

### Docker

```bash
docker run -p 8080:8080 jenkins/jenkins:lts
```


### Windows

使用 Jenkins 的 [Windows 安装程序](https://www.jenkins.io/download/) 


### Linux

通过包管理器安装 Jenkins（例如 [Apt](https://pkg.jenkins.io/debian-stable/)、[Yum](https://pkg.jenkins.io/redhat-stable/) 等）。


## 配置

安装完成后，通过浏览器访问 `http://localhost:8080` 进入 Jenkins 的管理界面。

## 创建任务

新建一个流水线（Pipeline）任务，编写 Jenkinsfile 定义构建步骤。


