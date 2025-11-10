MSBuild（Microsoft Build Engine）是用于构建 .NET 和 C++ 项目的平台和工具。它用于自动化构建过程，包括编译源代码、创建可执行文件或库、生成资源文件、运行单元测试、创建安装包等。

MSBuild 是 Visual Studio 内部的构建引擎，但也可以独立运行，不依赖于 Visual Studio。


**核心概念**：

1. **项目文件 (.csproj/.vcxproj)**：MSBuild 使用基于 XML 的项目文件来定义构建过程，通常以 `.csproj`（C#）或 `.vcxproj`（C++）作为扩展名。
2. **Targets（目标）**：构建项目时需要执行的一系列任务集合。MSBuild 可以有多个 Target，彼此之间可以有依赖关系。
3. **Tasks（任务）**：MSBuild 的基本操作单位，每个 Task 代表某种操作，比如编译代码、复制文件、生成文档等。
4. **Properties（属性）**：MSBuild 项目中的全局或局部变量，用于控制构建过程中的参数，例如输出路径、配置模式等。
5. **Items（项）**：MSBuild 项目中的输入文件和数据，比如源代码文件、资源文件等。


**使用**：
- **通过 Visual Studio 使用**：当你在 Visual Studio 中构建一个项目时，实际上是调用了 MSBuild 来执行构建。
- **命令行使用**：MSBuild 可以通过命令行调用，无需启动 Visual Studio。使用 `msbuild <项目文件>` 命令可以在命令行中执行构建。


### 项目文件结构

1. 根元素（必须）：`Project`，包含整个项目定义和命名空间引用
```xml
<?xml version="1.0" encoding="utf-8"?>
<Project DefaultTargets="Build" ToolsVersion="17.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
</Project>
```

2. 配置相关（必须）：`PropertyGroup`，包含项目属性的容器
	- `Configuration`：构建配置（Debug/Release）
	- `Platform`：目标平台（Win32/x64/ARM64）
	- `PlatformToolset`：编译器工具集（v143/v142/v141...）
	- `CharacterSet`：字符集（Unicode/MultiByte）
	- `WholeProgramOptimization`：全程序优化（true/false）

3. 项目文件定义
	- `ItemGroup`：项目项（如源文件）的容器
		- `ClCompile`：C/C++ 源文件
		- `ClInclude`：C/C++ 头文件
		- `Compile`：C# 源文件
		- `ResourceCompile`：资源文件
		- `CustomBuild`：自定义构建步骤的文件
		- `ProjectReference`：项目引用
		- `PackageReference`：NuGet 包引用
		- `None`：不参与构建的文件
```xml
  <ItemGroup>
    <ClCompile Include="main.cpp" />
    <ClCompile Include="utils.cpp" />
  </ItemGroup>
  
  <ItemGroup>
    <ClInclude Include="utils.h" />
  </ItemGroup>
```


### 命令行使用

通过命令行使用 MSBuild，可以灵活地控制项目的构建过程，尤其是在需要自动化构建或在 CI/CD 系统中时（如 Jenkins 或 GitLab CI）。

#### 查找 MSBuild.exe

如果未设置环境变量，手动查找 `MSBuild.exe` 的路径，在安装了 Visual Studio 的系统上，MSBuild 通常位于类似以下路径中：
- **Visual Studio 2019**: `C:\Program Files (x86)\Microsoft Visual Studio\2019\<Edition>\MSBuild\Current\Bin\MSBuild.exe`
- **Visual Studio 2022**: `C:\Program Files\Microsoft Visual Studio\2022\<Edition>\MSBuild\Current\Bin\MSBuild.exe`

#### 运行 MSBuild 命令

在命令行中，使用 MSBuild 来构建项目或解决方案文件：
```
msbuild project.csproj /t:Clean;Build /p:Configuration=Release /p:Platform=x64 
```

#####  常见参数

- 目标相关参数
	- `/t[arget]:<value>`              ：指定要执行的构建目标，多个目标用分号分隔
		- `Clean`：清理项目
		- `Build`：构建项目
		- `Rebuild`：清理并重新构建
		- `Restore`：
		- `GenerateProject`：生成项目文件
	- `/targets`                      ：显示指定项目文件中可用的目标列表

- 属性相关参数
	- `/p[roperty]:<name>=<value>`      ：设置或覆盖项目级属性
		- `Configuration`：Debug/Release
		- `Platform`：AnyCPU/x86/x64
		- `OutputPath`：输出目录
		- `TargetFramework`：目标框架

- 日志和输出参数
	- `/v[erbosity]:<level>`            ：指定输出详细程度
		- `q[uiet]`：安静模式
		- `m[inimal]`：最小输出（默认）
		- `n[ormal]`：标准输出
		- `d[etailed]`：详细输出
		- `diag[nostic]`：最详细输出
		- `/l[ogger]:<logger>`：指定构建记录器
		- `/nologo`：不显示启动版权信息
		- `/noconlog`：禁用控制台记录器

- 执行控制参数
	- `/m[axcpucount][:n]`              ：指定并行构建的最大CPU数量
	- `/nr:false`                      ：禁用节点重用
	- `/restore`                       ：在构建之前还原包依赖项

- 项目/解决方案相关参数
	- `/p:SolutionDir`                  ：指定解决方案目录
	- `/p:BuildInParallel=true`          ：并行构建多个项目

- 高级参数
	- `/noautoresponse`                 ：不自动包含MSBuild.rsp文件
	- `/preprocess[:file]`              ：创建包含所有导入文件的单个预处理项目文件
	- `/detailedsummary`                ：显示构建的详细摘要信息

- 查看 MSBuild 帮助
	- `/?`                            ：查看 MSBuild 的帮助信息
```bash
msbuild /?
```


### CUDA 相关

1. **安装 CUDA Toolkit**：确保你已经安装了 NVIDIA 的 CUDA Toolkit，并配置好了环境变量。通常会安装到类似以下路径：
    - Windows: `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\vX.X`
    - Linux: `/usr/local/cuda-X.X`
2. **安装 Visual Studio**：确保你使用的是支持 CUDA 的 Visual Studio 版本，因为 CUDA 与 Visual Studio 编译器有较强的兼容性要求。通常 CUDA 会与 Visual Studio 配合使用，安装时会自动关联。
3.  **配置环境变量**：
    - 将 CUDA 的 `bin` 和 `lib` 路径添加到系统的环境变量中：
        - `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\vX.X\bin`
        - `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\vX.X\lib\x64`

4. **CUDA 文件的配置**：CUDA 文件通常有 `.cu` 扩展名，在 `.vcxproj` 项目文件中，MSBuild 会根据文件扩展名自动识别 CUDA 文件，并使用 `nvcc` 进行编译（对于 GPU 部分直接编译，对于 CPU 部分通过 MSBuild 调用 Visual Studio 的 C++ 编译器编译）。
5. **编译包含 CUDA 的项目**：MSBuild 会根据 `.vcxproj` 文件中的设置，调用 `nvcc` 编译 `.cu` 文件，同时处理其余的 C++ 源文件。
```bash
msbuild MyProject.sln /p:Configuration=Release /p:Platform=x64
```


