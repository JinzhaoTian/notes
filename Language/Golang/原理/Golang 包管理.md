Go 语言的包管理经历了几个发展阶段，现在的标准方式是使用 Go Modules。

## 当前标准：Go Modules

自 Go 1.11 引入，1.16 成为默认模式。

### 核心文件

1. **`go.mod`**：模块定义文件
2. **`go.sum`**：依赖校验文件

### 基本命令

```bash
# 初始化模块
go mod init <module-path>

# 添加依赖
go get <package>[@version]

# 整理依赖
go mod tidy

# 下载依赖到本地缓存
go mod download

# 查看依赖图
go mod graph
```

### `go.mod` 示例

```go
module github.com/username/project

go 1.20

require (
    github.com/gin-gonic/gin v1.9.0
    golang.org/x/sync v0.3.0
)

replace github.com/old/pkg => ./local/pkg
```


## 依赖版本管理

### 版本规则

```bash
# 获取最新版本
go get example.com/pkg

# 获取指定版本
go get example.com/pkg@v1.2.3

# 获取特定commit
go get example.com/pkg@master
go get example.com/pkg@abcd1234
```

### 版本选择

1. 语义化版本（SemVer）：`v1.2.3`
2. Go 会自动选择满足要求的最低兼容版本
3. 使用 `go list -m all` 查看实际使用的版本


## **代理和私有仓库**

### 配置镜像代理

```bash
# 设置 GOPROXY
go env -w GOPROXY=https://goproxy.cn,direct
go env -w GOPRIVATE=*.company.com,github.com/private
```

### 常用代理

1. 官方：`https://proxy.golang.org`
2. 阿里云：`https://mirrors.aliyun.com/goproxy/`
3. 七牛云：`https://goproxy.cn`

## vendor 目录（可选）

Vendor 是 Go 中用来存储项目依赖包副本的目录，通常位于项目根目录下的 `vendor/` 文件夹中。

```
my-project/
├── go.mod
├── go.sum
├── main.go
└── vendor/           # 所有依赖的本地副本
    ├── github.com/
    │   └── gin-gonic/
    │       └── gin/
    └── golang.org/
        └── x/
            └── sync/
```


### 基本命令

```bash
# 将依赖复制到 vendor 目录
go mod vendor

# 使用 vendor 构建
go build -mod=vendor
```




## 工作区模式（Go 1.18+)

用于多模块开发：
```bash
# 初始化工作区
go work init ./module1 ./module2

# 添加模块到工作区
go work use ./module3
```

**`go.work` 示例**：
```go
go 1.20

use (
    ./module1
    ./module2
    ../external/module
)
```


## **最佳实践**

### 项目结构示例
```
my-project/
├── go.mod
├── go.sum
├── main.go
├── internal/
│   └── utils/
├── pkg/
│   └── api/
├── cmd/
│   ├── server/
│   └── cli/
└── vendor/ (可选)
```

### 常用工作流

```bash
# 新项目初始化
go mod init github.com/username/project
go mod tidy

# 日常开发
go get -u <package>      # 更新包
go get -u ./...          # 更新所有依赖
go mod tidy              # 清理无用依赖

# 版本发布
go mod tidy
go test ./...
go build -ldflags="-s -w"
```