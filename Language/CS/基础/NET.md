.NET 是由微软开发的一个跨平台、开源的开发框架，用于构建各种类型的应用程序（如 Web、桌面、移动、云服务等）。

![](imgs/Pasted%20image%2020240312104327.png)

.NET Core 是一个可以用来构建现代、可伸缩和高性能的跨平台软件应用程序的通用开发框架，可用于为 Windows、Linux 和 macOS 构建软件应用程序。.NET Core 是最通用的框架，可用于构建各种软件，包括Web应用程序、移动应用程序、桌面应用程序、云服务、微服务、API、游戏和物联网应用程序。

.NET Core 并不局限于单一的编程语言，它支持 C#、VB.NET、F#、XAML 和 TypeScript，这些编程语言都是开源的，由独立的社区管理。

![](imgs/Pasted%20image%2020240312100346.png)

`.NET Core 3.1` 是 .NET Core 的最后一个版本，`.NET 5`（2020年） 开始，微软统一了 .NET Framework 和 .NET Core，形成 "`.NET`"（无 "Core" 后缀），成为未来所有 .NET 应用的统一平台。


### .NET CLI

`dotnet` 命令是 .NET CLI（命令行接口）的核心工具，用于开发、构建、运行和管理 .NET 应用程序。

1. **`dotnet new`** ：**创建新项目或文件**（如控制台、Web API、类库等）
```bash
dotnet new <TEMPLATE> [--dry-run] [--force] [-lang|--language {"C#"|"F#"|VB}]
    [-n|--name <OUTPUT_NAME>] [-f|--framework <FRAMEWORK>] [--no-update-check]
    [-o|--output <OUTPUT_DIRECTORY>] [--project <PROJECT_PATH>]
    [-d|--diagnostics] [--verbosity <LEVEL>] [Template options]
```
- `<TEMPLATE>`
	- `console` ：控制台应用程序
	- `wpf` ：WPF 应用程序
	- `webapi` ：ASP.NET Core Web API


2. **`dotnet build`** ：**编译项目及其依赖项**。
```bash
dotnet build [<PROJECT>|<SOLUTION>] [-a|--arch <ARCHITECTURE>]
    [--artifacts-path <ARTIFACTS_DIR>]
    [-c|--configuration <CONFIGURATION>] [-f|--framework <FRAMEWORK>]
    [--disable-build-servers]
    [--force] [--interactive] [--no-dependencies] [--no-incremental]
    [--no-restore] [--nologo] [--no-self-contained] [--os <OS>]
    [-o|--output <OUTPUT_DIRECTORY>]
    [-p|--property:<PROPERTYNAME>=<VALUE>]
    [-r|--runtime <RUNTIME_IDENTIFIER>]
    [--self-contained [true|false]] [--source <SOURCE>]
    [--tl:[auto|on|off]] [--use-current-runtime, --ucr [true|false]]
    [-v|--verbosity <LEVEL>] [--version-suffix <VERSION_SUFFIX>]
```
- `<PROJECT> | <SOLUTION>`
	- 要生成的项目或解决方案文件，如果未指定项目或解决方案文件，MSBuild 会在当前工作目录中搜索文件扩展名以 `proj` 或 `sln` 结尾的文件并使用该文件。


3. **`dotnet run`** ：**编译并立即运行项目**（默认调试模式）。
```bash
dotnet run [-a|--arch <ARCHITECTURE>] [-c|--configuration <CONFIGURATION>]
    [-f|--framework <FRAMEWORK>] [--force] [--interactive]
    [--launch-profile <NAME>] [--no-build]
    [--no-dependencies] [--no-launch-profile] [--no-restore]
    [--os <OS>] [--project <PATH>] [-r|--runtime <RUNTIME_IDENTIFIER>]
    [--tl:[auto|on|off]] [-v|--verbosity <LEVEL>]
    [[--] [application arguments]]
```

4. **`dotnet package add`** ：**添加 NuGet 包引用**。
```bash
dotnet package add <PACKAGE_NAME>
    [-f|--framework <FRAMEWORK>] [--interactive] [--project <PROJECT>]
    [-n|--no-restore] [--package-directory <PACKAGE_DIRECTORY>]
    [--prerelease] [-s|--source <SOURCE>] [-v|--version <VERSION>]
```

```bash
dotnet package add Newtonsoft.Json  # 添加 Newtonsoft.Json 包
```

5. **`dotnet package remove`** ：**移除 NuGet 包引用**。
```bash
dotnet package remove <PACKAGE_NAME> [--project <PROJECT>]
```

```bash
dotnet package remove Newtonsoft.Json
```



