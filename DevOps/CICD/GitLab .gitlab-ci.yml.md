
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

## 参数详解

### `stages`

`stages` 定义了流水线中全局可用的各个阶段，阶段元素的排序定义了作业执行的顺序，通常包含诸如构建、测试和部署等步骤。

1. 相同 `stage` 的作业并行运行。
2. 默认情况下，`stage` 有三个：`build` 、`test`、`deploy`
	- 如果一个作业未定义 `stage` 阶段，则作业使用 `test`
3. 默认情况下，上一个 `stage` 的作业全部运行成功后才执行下一个 `stage` 的作业。
	- 任何一个前置的作业失败了，`commit` 提交会标记为 `failed` 并且下一个 `stage` 的作业都不会执行。

### `jobs`

> [!tip]
> 实际没有这个关键字 `jobs` 作业，通常自定名称。

作业 `job` 是 GitLab CI/CD 中的基本执行单元，每个 `job` 对应一个特定的任务，通常包含编译、测试或部署步骤。

```yaml
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
```

#### 字段

1. `stage`：指定作业所属的 `stages`
2. `tags`：匹配 GitLab Runner，用于将特定的 Runner 分配给特定任务，确保任务运行在合适的环境中。
	-  任务的 `tags` 必须与 Runner 的标签相匹配，任务才能分配到该 Runner 上执行。
3. `before_script`：在所有任务开始前运行的命令，可以为流水线设置通用的初始化步骤。
4. `script` ：是任务最重要的字段，用于定义执行的命令或脚本。每个任务都会在指定的执行环境中依次运行这些命令。
	- 命令按顺序执行，如果某个命令失败，任务就会失败。
5. `after_script`：所有任务结束后（成功或失败）运行的命令，通常用于清理资源或生成日志等操作。
6. `only`：指定在特定的分支、标签或条件下执行任务。
7. `artifacts`：工件，或者归档文件。
	- `name`：工件的默认名称是 `artifacts`，当下载时名称是 `artifacts.zip`，可用该关键字自定义工件的归档名称。
	- `paths`：用于指定任务生成的文件或目录，仅仅项目工作空间（`project workspace`）的路径可以使用。
	- `exclude`：排除文件
	- `when`：用于在作业失败时或者忽略失败时上传工件。
		- `on_success`，默认值，当作业成功上传工件。
		- `on_failure`，当作业失败上传工件。
		- `always`，无论作业是否成功一直上传工件。
	- `expire_in`：设置工件的过期时间。
8. `dependencies` ：用于指定当前任务依赖的其他任务的 artifacts，确保当前任务可以访问这些 artifacts。
9. `image` ：用于在 Docker 环境下指定任务运行的基础镜像。

### `variables`

`variables` ：定义环境变量，可以在任务中使用这些变量。

> [!caution] 敏感变量在 GitLab 的 UI 中设置 CI/CD 变量
> 直接在 `.gitlab-ci.yml` 文件的 `variables` 区块下硬编码敏感信息是非常不安全的做法，这些变量（尤其是密码）会以明文形式存储在代码仓库中，任何有仓库访问权限的人都能看到，存在严重的安全风险。

> [!tip] CI/CD 变量设置
> 在 GitLab 项目页面 UI，依次点击 `Settings` -> `CI/CD` > `Variables`，
> 1. 添加变量的 `Key` 和 `Value`
> 2. 确定 `Environment Scope`：
> 	- 选择 All (default) (`*`)
> 	- 或者指定环境
> 3. **确定保护状态**：
> 	- 对于受保护分支，确保变量要勾选 `"Protect variable` 选项；
> 	- 对于没有被设置为受保护分支，确保变量没有勾选 `"Protect variable"` 选项）。

> [!caution] 受保护变量默认只对受保护分支的流水线可见
> 如分支受保护，那么设置的分支流水线变量也要受保护；如果暂时不想保护分支，需要取消变量 `"Protect variable` 的选项，这样变量对所有分支的流水线都可见，手动触发时也会被直接传入。


