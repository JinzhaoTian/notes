不应该一个文件一个文件的看，应该是首先能成功编译，在运行中摸索运行逻辑。

## 源码下载

获取：[EpicGames/UnrealEngine: Unreal Engine source code](https://github.com/EpicGames/UnrealEngine)

## 构建

### Windows

1. **环境准备**
2. **生成项目文件**
	- 双击 `Setup.bat`
	- 双击 `GenerateProjectFiles.bat`，会生成 `UE5.sln` 文件
3. **编译引擎**
	- 用 `VS 2022` 版本打开，点击安装额外组件，![](_imgs/Pasted%20image%2020251120093042.png)
	- 将 UE5 设为启动项，![](_imgs/Pasted%20image%2020251120092912.png)
	- 设置正确的调试模式，如 `Development Editor and your solution platform to Win64` 等待**编译 40 分钟左右**即可。


### macOS

1. **环境准备**
	- **Xcode 版本**：建议使用 Xcode 16.0 或 16.1。
2. **生成项目文件**
	- 进入 UnrealEngine 源码目录。
	- 运行 `./Setup.command` 脚本，这个脚本会下载编译所需的依赖项，下载量可能较大（约11GB），需要耐心等待。
	- 运行 `./GenerateProjectFiles.command` 脚本，这将在当前目录生成 `UE5.xcworkspace` 文件，用于在 Xcode 中打开和编译
3. **设置目标架构**
	-  M4 芯片属于 ARM64 架构，`-architecture=arm64`
4. **编译引擎**
	- 用 Xcode 打开生成的 `UE5.xcworkspace`。
	- 在 Xcode 的 scheme 列表中，**首先需要单独编译 `ShaderCompileWorker` 目标**。这个工具负责着色器的离线编译，必须优先构建。
	- `ShaderCompileWorker` 编译成功后，再将目标切换为 `UE5Editor`（如果你需要编译编辑器）或你游戏的项目进行编译。整个编译过程非常耗时，且对硬件要求高，请确保 Mac 有良好的散热。

## 脚本解析


### `GenerateProjectFiles.[bat|sh|command]`

`GenerateProjectFiles.[bat|sh|command]` 是 Unreal Engine 一个关键脚本文件，其核心作用是为引擎的源代码生成所需的 IDE 项目文件（例如 Visual Studio 的 `.sln` 文件， Xcode 的 `.xcodeproj` 文件）。

当获取了 Unreal Engine 的源代码后，需要运行这个脚本，它会分析引擎内各个模块的依赖关系，创建出一个结构正确、能够顺利编译和调试的 IDE 项目，然后在 IDE 里编译。

#### 工作流程

1. **定位与分析**：脚本会遍历引擎目录，读取每个模块的 `.build.cs` 文件
2. **处理依赖**：确定所有模块之间的依赖关系、所需的库文件、包含路径等编译信息
3. **生成项目**：根据分析结果，生成与你系统上安装的开发环境相匹配的项目文件。


