
## .NET CLI 内置核心命令

| 类别          | 命令                      | 主要功能                             |
| ----------- | ----------------------- | -------------------------------- |
| **项目与程序操作** | `dotnet new`            | 根据模板（如 `console`, `webapi`）创建新项目 |
|             | `dotnet build`          | 编译项目及依赖项，生成可执行文件或库               |
|             | `dotnet run`            | 编译并立即运行项目（无需显式先`build`）          |
|             | `dotnet test`           | 运行项目中定义的单元测试                     |
|             | `dotnet publish`        | 将应用程序及其依赖项打包，用于部署                |
|             | `dotnet clean`          | 清理项目的生成输出文件（如`bin`， `obj`目录）     |
| **依赖与包管理**  | `dotnet restore`        | 还原项目文件（`.csproj`）中指定的依赖包         |
|             | `dotnet add package`    | 向项目添加 NuGet 包引用                  |
|             | `dotnet add reference`  | 添加项目到项目（P2P）的引用                  |
|             | `dotnet remove package` | 从项目中移除 NuGet 包引用                 |
|             | `dotnet nuget`          | 提供一系列 NuGet 源和包管理命令              |
| **工具管理**    | `dotnet tool install`   | 全局或本地安装 .NET 工具                  |
|             | `dotnet tool update`    | 更新已安装的工具                         |
|             | `dotnet tool uninstall` | 卸载已安装的工具                         |
|             | `dotnet tool list`      | 列出所有已安装的工具                       |
| **系统与环境**   | `dotnet --info`         | 显示 .NET SDK、运行时版本及系统环境的详细信息      |
|             | `dotnet --list-sdks`    | 列出所有已安装的 .NET SDK 版本             |
|             | `dotnet --help`         | 显示所有可用命令的概览或特定命令的详细帮助            |

### `dotnet new`

```
```


### `dotnet build`

```bash
dotnet build [<PROJECT>|<SOLUTION>|<FILE>] [-a|--arch <ARCHITECTURE>]
    [--artifacts-path <ARTIFACTS_DIR>]  [-bl|--binaryLogger:<FILE>]
    [-c|--configuration <CONFIGURATION>] [--disable-build-servers]
    [-f|--framework <FRAMEWORK>] [--force] [--interactive]
    [--no-dependencies] [--no-incremental] [--no-restore] [--nologo]
    [--no-self-contained] [-o|--output <OUTPUT_DIRECTORY>] [--os <OS>]
    [-p|--property:<PROPERTYNAME>=<VALUE>] [-r|--runtime <RUNTIME_IDENTIFIER>]
    [--sc|--self-contained] [--source <SOURCE>]
    [--tl:[auto|on|off]] [ --ucr|--use-current-runtime]
    [-v|--verbosity <LEVEL>] [--version-suffix <VERSION_SUFFIX>]

dotnet build -h|--help
```



## dotnet tool 扩展工具


