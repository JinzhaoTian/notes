GitLab CI 是 [GitLab](../Code%20Version/GitLab.md) 提供内置的 CI/CD 管道功能，开发者可以通过 `.gitlab-ci.yml` 配置文件定义自动化构建、测试和部署流程。当有代码提交时，GitLab 自动执行这些流程。

![](_imgs/Pasted%20image%2020240918151929.png)

在 GitLab CI 中 必须设置 Runner 才能运行 CI/CD 流水线的任务。

### 配置文件

```yaml
stages:
  - build
  - test
  - deploy

variables:
  CUDA_PATH: "C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v11.0"
  MSBUILD_PATH: "C:/Program Files (x86)/Microsoft Visual Studio/2019/Community/MSBuild/Current/Bin/MSBuild.exe"

build-job:
  stage: build
  tags: 
    - windows
  before_script:
    - echo "This runs before any job"
  script:
    - echo "Building project"
    - make build
    - '$MSBUILD_PATH path_to_your_project.sln /p:Configuration=Release'
  after_script:
    - echo "This runs after all jobs"
  artifacts: 
    paths: 
      - binaries/
  only:
    - master 
    - tags

test-job:
  stage: test
  script:
    - echo "Testing project"
    - make test
  dependencies: 
    - build-job
```

#### 关键字

1. `stages` ：定义了流水线中的各个阶段，阶段是任务（jobs）执行的顺序单元，通常包含诸如构建、测试和部署等步骤。
	- 任务按阶段顺序依次执行，前一个阶段成功后才会执行下一个阶段。

2. `jobs` ：实际没有这个关键字，可以自定名称。是 GitLab CI/CD 中的基本执行单元，每个 `job` 对应一个特定的任务，通常包含编译、测试或部署步骤。
	1. `script` ：是任务最重要的字段，用于定义执行的命令或脚本。每个任务都会在指定的执行环境中依次运行这些命令。
		- 命令按顺序执行，如果某个命令失败，任务就会失败。
	2. `before_script`：在所有任务开始前运行的命令，可以为流水线设置通用的初始化步骤。
	3. `after_script`：所有任务结束后（成功或失败）运行的命令，通常用于清理资源或生成日志等操作。
	4. `tags` ：匹配 Runner，用于将特定的 Runner 分配给特定任务，确保任务运行在合适的环境中。
		-  任务的 `tags`必须与 Runner 的标签相匹配，任务才能分配到该 Runner 上执行。
	5. `only` ：指定在特定的分支、标签或条件下执行任务。
	6. `artifacts` ：用于定义任务生成的文件或目录，这些文件会在任务完成后保存并可以在后续任务中使用。
	7. `dependencies` ：用于指定当前任务依赖的其他任务的 artifacts，确保当前任务可以访问这些 artifacts。
	8. `image` ：用于在 Docker 环境下指定任务运行的基础镜像。

3. `variables` ：定义环境变量，可以在任务中使用这些变量。



### GitLab Runner

GitLab Runner 是 GitLab CI/CD 中用于执行任务的应用程序，是 GitLab CI/CD 流水线的核心组件，负责在不同环境中运行在 `.gitlab-ci.yml` 文件中定义的构建、测试、部署等任务，没有 Runner，GitLab 无法执行任何任务。

#### 使用

1. **安装 GitLab Runner**：可以在各种操作系统上安装 GitLab Runner，包括 Linux、macOS 和 Windows，参考[文档](https://docs.gitlab.com/runner/install/)。
2. **注册 GitLab Runner**：安装完 Runner 后，需要在 Runner 所在的服务器上运行注册命令，并提供 GitLab 实例的 URL 和项目或组的注册令牌。执行 `gitlab-runner register` 命令时，会让你选择执行器并设置标签。
3. **使用标签**：当你在 `.gitlab-ci.yml` 文件中定义任务时，可以使用 `tags` 字段指定使用哪些 Runner。每个 Runner 注册时可以定义一些标签，GitLab 会根据标签匹配合适的 Runner 来执行任务。