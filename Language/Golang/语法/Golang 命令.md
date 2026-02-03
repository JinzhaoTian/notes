## 基础命令

1. **`go run`**：编译并运行
```bash
go run main.go          # 编译并运行 Go 程序
go run .                # 运行当前目录
go run ./cmd/app        # 运行指定目录
```

2. **`go build`**：编译
```bash
go build                # 编译当前目录
go build -o app         # 指定输出文件名
go build ./cmd/server   # 编译特定包
go build -ldflags="-s -w"  # 减小二进制大小
```

3. **`go install`**：编译并安装
```bash
go install              # 编译并安装到 $GOPATH/bin
go install ./...        # 安装当前项目所有包
```

4. **`go test`**：运行测试
```bash
go test                 # 运行测试
go test -v              # 详细输出
go test -race           # 检测数据竞争
go test -cover          # 测试覆盖率
go test ./...           # 测试所有子目录
```

5. **`go get`**：下载依赖
```bash
go get github.com/pkg/example  # 下载依赖包
go get -u              # 更新依赖
go get -u all          # 更新所有依赖
```

6. **`go mod`** (模块管理)
```bash
go mod init example.com/app    # 初始化模块
go mod tidy            # 整理依赖
go mod download        # 下载依赖
go mod vendor          # 创建 vendor 目录
go mod graph           # 显示依赖图
go mod verify          # 验证依赖完整性
```

## 开发相关命令

7. **`go fmt`**：格式化
```bash
go fmt ./...           # 格式化所有 Go 文件
```

8. **`go vet`**
```bash
go vet ./...           # 代码静态分析
```

9. **`go generate`**
```bash
go generate ./...      # 执行代码生成指令
```

10. **`go doc`**
```bash
go doc fmt.Println     # 查看文档
go doc -all fmt        # 查看包所有文档
```

11. **`go list`**
```bash
go list ./...          # 列出所有包
go list -m all         # 列出所有依赖
go list -json          # JSON 格式输出
```

## 工具命令

12. **`go tool`**
```bash
go tool pprof          # 性能分析
go tool trace          # 跟踪分析
go tool cover          # 覆盖率工具
go tool compile        # 编译器
go tool link           # 链接器
```

13. **`go clean`**
```bash
go clean               # 清理编译文件
go clean -cache        # 清理缓存
go clean -testcache    # 清理测试缓存
```

14. **`go env`**
```bash
go env                 # 显示所有环境变量
go env GOPATH          # 显示特定变量
go env -w GOPROXY=https://goproxy.cn  # 设置环境变量
```

## 实用组合命令

1. **开发和测试**
```bash
# 开发时常用组合
go mod tidy && go fmt ./... && go vet ./... && go test ./...

# 构建并运行
go build && ./app

# 交叉编译
GOOS=linux GOARCH=amd64 go build
```

2. **性能分析**
```bash
# 生成 CPU 剖析文件
go test -cpuprofile=cpu.prof -bench=.

# 生成内存剖析文件
go test -memprofile=mem.prof -bench=.

# 分析剖析文件
go tool pprof cpu.prof
```

3. **版本信息**
```bash
go version            # 查看 Go 版本
go version -m app     # 查看二进制文件的构建信息
```

## 常用环境变量设置

```bash
# 设置代理（国内推荐）
go env -w GOPROXY=https://goproxy.cn,direct

# 禁用模块或私有仓库验证
go env -w GOSUMDB=off
go env -w GOPRIVATE=*.corp.example.com

# 开启 Go Modules
go env -w GO111MODULE=on
```


## 小贴士

1. **查看命令帮助**：
```bash
go help
go help build
go help modules
```

