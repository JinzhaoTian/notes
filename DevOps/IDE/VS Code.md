挺好的**编辑器**，强大的插件系统，跨平台。

> [!info] Visual Studio Code
Visual Studio Code is a lightweight but powerful source code editor which runs on your desktop and is available for Windows, macOS and Linux. It comes with built-in support for JavaScript, TypeScript and Node.js and has a rich ecosystem of extensions for other languages and runtimes (such as C++, C#, Java, Python, PHP, Go, .NET). 

![](_imgs/Pasted%20image%2020240112153048.png)


## C++ 开发配置

### 安装 C++ 编译器

#### [Clang](../../Language/C++/构建/Clang.md)

1. **Windows**：可以通过 Visual Studio 安装，或者通过 LLVM [官方安装包](https://github.com/llvm/llvm-project/releases)安装
	- 双击下载的 `.exe` 文件，按照向导安装
	- 勾选 "Add LLVM to the system PATH"
	- 验证安装：
```bash
clang --version
```

2. **macOS**：默认安装了 `Clang`（Xcode 工具链的一部分），但建议安装完整的 Xcode 命令行工具：
```bash
xcode-select --install
```

3. **Linux**：可以通过[系统包管理器](../../Operation%20System/Linux/Linux%20系统包管理器.md)安装
```bash
# Ubuntu/Debian
sudo apt install clang
```

```bash
# Fedora/RHEL/CentOS
sudo yum install clang
```


#### [GCC](../../Language/C++/构建/GCC.md)

1. **Windows**：可以安装 GCC 在 Windows 下的移植版本 [MinGW](../../Language/C++/构建/MinGW.md) 。

2. **macOS**：可以通过 Homebrew 安装 GCC ：
```bash
brew install gcc
```

3. **Linux**：通过系统包管理器安装
```bash
# Ubuntu/Debian
sudo apt install gcc g++ build-essential
```

```bash
# CentOS 7
sudo yum install gcc gcc-c++
```

#### [MSVC](../../Language/C++/构建/MSVC.md)

Windows 下 Visual Studio 自带的 C/C++ 编译器。

### 安装 C/C++ 插件

1. 安装 `C/C++`（Microsoft 官方扩展 `ms-vscode.cpptools`，提供智能提示、调试支持）
![](_imgs/Pasted%20image%2020250714102841.png)

2. 安装 `CMake Tools`（Microsoft 官方扩展 `ms-vscode.cmake-tools`）
![](_imgs/Pasted%20image%2020250714110615.png)

3. （可选）安装 `clang` （LLVM 官方扩展 `llvm-vs-code-extensions.vscode-clangd`）
![](_imgs/Pasted%20image%2020250717104324.png)



#### `c_cpp_properties.json`

`C/C++` 的配置文件 `c_cpp_properties.json` ，用于控制 C/C++ 代码的智能感知（IntelliSense）、代码导航、错误检查等核心功能。它的作用是为项目定义编译器路径、包含路径（include paths）、宏定义（defines）、编译标准（C++ 版本）等关键信息。

```json
{
	"configurations": [
		{
			"name": "Linux",  // 配置名称（如 Win32、Mac、Linux）
			"includePath": [  // 头文件搜索路径
				"${workspaceFolder}/**",
				"/usr/include/**"
			],
			"defines": ["DEBUG=1"],  // 预定义宏
			"compilerPath": "/usr/bin/g++",  // 编译器路径
			"cStandard": "c17",      // C 语言标准
			"cppStandard": "c++17",  // C++ 语言标准
			"intelliSenseMode": "linux-gcc-x64",  // 智能感知模式
			"compileCommands": "${workspaceFolder}/build/compile_commands.json"  // 编译数据库
		}
	],
	"version": 4
}
```

**关键字段详解**：
1. `includePath` ：指定头文件（`.h`/`.hpp`）的搜索路径，支持通配符 `**`（递归匹配）
2. `defines` ：预定义的宏（相当于 `-D` 编译选项），用于条件编译
3. `compilerPath` ：指定编译器路径（如 `g++`、`clang++`、`msvc`），用于推断系统包含路径和默认编译标准
4. `cStandard` / `cppStandard` ：设置语言标准
	- `c11`
	- `c17`
	- `c++20`
5. `intelliSenseMode` ：指定智能感知引擎的模式，需与编译器匹配：
	- `linux-gcc-x64`
	- `windows-msvc-x64`
	- `macos-clang-arm64`
6. `compileCommands` ：指向 CMake 生成的 `compile_commands.json` 文件，自动同步编译选项（优先级高于手动配置的 `includePath` 和 `defines`）


#### CMake Tools






### 配置 C++ 环境

#### 步骤 1：创建项目文件夹

用 VS Code 打开一个空文件夹（如 `~/cpp_project`）。

#### 步骤 2：编写代码

新建 `main.cpp` 文件，写入：

```cpp
#include <iostream>
using namespace std;

int main() {
    cout << "Hello, C++ on macOS!" << endl;
    return 0;
}
```

#### 步骤 3：配置编译器路径

1. 按 `Cmd+Shift+P` 输入 **C/C++: Edit Configurations (UI)**，打开配置界面。
2. 在 **Compiler path** 中指定编译器路径
3. 保存后会在 `.vscode/c_cpp_properties.json` 中生成配置。

#### 步骤 4：配置构建任务

1. 按 `Cmd+Shift+P` 输入 **Tasks: Configure Task**，选择 **Create tasks.json**。
2. 选择 **Others**，然后修改文件为：
```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Build C++",
            "type": "shell",
            "command": "g++", // clang++, MSVC
            "args": [
                "-std=c++17",
                "-g",
                "${file}",
                "-o",
                "${fileDirname}/${fileBasenameNoExtension}"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "problemMatcher": ["$gcc"]
        }
    ]
}
```

#### 步骤 5：配置调试

1. 点击左侧调试图标（或 `Cmd+Shift+D`），选择 **Create a launch.json file**。
2. 选择 **C++ (GDB/LLDB)**，修改配置：
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug C++",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}/${fileBasenameNoExtension}",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "lldb",
            "preLaunchTask": "Build C++"
        }
    ]
}
```


#### 步骤 6：智能感知

**方法 1（推荐）**：通过 `compile_commands.json` 

1. 在 `CMakeLists.txt` 中设置：
```cmake
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
```

2. 在 `.vscode/settings.json` 中添加：
```json
{
    "C_Cpp.default.compileCommands": "${workspaceFolder}/build/compile_commands.json",
    "C_Cpp.intelliSenseEngine": "Default",

	// 另一种方式设置cmake
	"cmake.configureSettings": {
        "CMAKE_EXPORT_COMPILE_COMMANDS": "ON"  # 确保CMake生成该文件
    }
}
```

**方法 2（备用）**：手动添加包含路径，在 `.vscode/c_cpp_properties.json` 中：
```json
"includePath": [
    "${workspaceFolder}/**",
    "${workspaceFolder}/vcpkg_installed/x64-windows/include"
]
```


**方法 3**：自定义 IntelliSense 引擎，如使用 `clangd` 替代默认引擎：

1. 安装 `clangd` 扩展

2. 禁用默认 `C/C++` 扩展的 IntelliSense：
```json
{
	"C_Cpp.intelliSenseEngine": "disabled"
}
```

3. `clangd` 会自动检测 `compile_commands.json`。







### 运行和调试

- **构建**：按 `Cmd+Shift+B` 编译代码。
- **运行**：按 `F5` 启动调试，或使用 **Code Runner** 扩展（快捷键 `Ctrl+Option+N`）。
- **调试**：在代码中设置断点，按 `F5` 进入调试模式。



## Java 开发配置

### 安装 JDK



### 安装 Java 插件

- **Java Extension Pack**（包含调试、智能提示等功能）
- **Language Support for Java™ by Red Hat**
- **Debugger for Java**


### 配置 Java 环境

#### 步骤 1：设置 JDK 路径

- 按 `Cmd + ,` 打开设置，搜索 `java home`
- 在 `settings.json` 中添加


#### 步骤 2：创建 Java 项目

- 按 `Cmd+Shift+P`，输入 `Java: Create Java Project`
- 选择 `No build tools`（或 Maven/Gradle）

#### 步骤 3：编写代码

创建 `Hello.java`：
```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello, Java!");
    }
}
```

#### 步骤 4：配置 Maven

1. 安装 Maven
2. 安装 **Maven for Java** 插件




### 运行和调试

- **运行**：按 `F5` 调试，或使用 **Code Runner** 插件（`Ctrl+Option+N`）运行。



## C# 开发设置

### 安装 .NET SDK

从[官网](https://dotnet.microsoft.com/zh-cn/download)下载，按向导完成安装。

验证安装：
```bash
dotnet --version  # 检查 SDK 版本
dotnet --list-sdks  # 查看已安装的 SDK
```

### 安装 C# 插件

- **C# Dev Kit**（官方扩展，提供智能提示、项目管理）
- **C# for Visual Studio Code**（基础语言支持）
- **.NET Core Extension Pack**（调试和工具集成）
- **vscode-solution-explorer**（管理 `.sln` 解决方案文件）



### 配置 C# 环境

#### 步骤 1：创建 C# 项目

方法 1：命令行创建：
```bash
mkdir MyCSharpProject
cd MyCSharpProject
dotnet new console  # 创建控制台项目
```

方法 2：创建解决方案（适用于多项目）：
```bash
dotnet new sln -n MySolution  # 创建解决方案
dotnet new console -o MyApp   # 创建控制台项目
dotnet sln add MyApp/MyApp.csproj  # 将项目加入解决方案
```


#### 步骤 2：配置调试

按 `F5` 或点击左侧调试图标，选择 **.NET Core** 环境，VS Code 会自动生成 `launch.json` 和 `tasks.json` 

`launch.json`：
```json
{
    "configurations": [
        {
            "name": ".NET Core Launch (console)",
            "type": "coreclr",
            "request": "launch",
            "program": "${workspaceFolder}/bin/Debug/<TARGET-FRAMEWORK>/<PROJECT-NAME>.dll",
            "args": [],
            "cwd": "${workspaceFolder}",
            "console": "internalConsole"
        }
    ]
}
```


#### 步骤 3：NuGet 包管理

安装 **Visual NuGet** 插件，右键 `.csproj` 文件选择 **Manage Packages** 管理依赖



### 运行和调试

- **编译运行**：
```bash
dotnet build  # 编译
dotnet run    # 运行
```

- **调试**：在代码中设置断点，按 `F5` 启动调试。


## Node 开发设置






