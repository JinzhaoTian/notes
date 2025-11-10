[FlaxEngine](https://github.com/FlaxEngine/FlaxEngine) 是一个开源的游戏引擎，支持 DirectX 12、Vulkan，专门为开发高性能的 3D 游戏和实时应用而设计。它提供了一套现代化的工具和功能，支持从小型独立游戏到大型商业项目的开发。


## 架构

### 目录结构

- **Binaries/** - 可运行文件
    - **Editor/** - Flax Editor 的二进制文件
    - **Tools/** - tools 的二进制文件
- **Cache/** - 引擎或者工具使用的本地缓存路径
    - **Intermediate/** - 引擎构建时的中间文件和缓存
        - **_ProjectName_/** - per-project build cache data
        - **Deps/** - Flax.Build dependencies building cache
    - **Projects/** - 项目文件地址，主要有 `Flax.vcxproj` 和 `BuildScripts.csproj`
- **Content/** - 引擎和编辑器所使用的 assets 和 binary files
- **Development/** - engine 开发文件
    - **Scripts/** - 必要脚本
- **Source/** - 源码路径
    - **Editor/** - 编辑器源码
    - **Engine/** - 引擎源码
    - **Platforms/** - 不同平台实现的源文件和依赖文件
        - **DotNet/** - C# 依赖文件
        - **Editor/** - Flax Editor 二进制文件
        - **_PlatformName_/** - per-platform files
            - **Binaries/** - per-platform binaries
                - **Game/** - Flax Game binaries
                - **ThirdParty/** - prebuilt 3rd Party binaries
    - **Shaders/** - shaders 源码
    - **ThirdParty/** - 第三方源码
    - **Tools/** - 项目生成工具源码
	    - **Flax.Build/**
		    - Flax.Build.csproj
	    - **Flax.Build.Tests/**

#### 编译逻辑

对于 Windows，
```
Flax.Build.csproj
      |
      |
  [MSBUILD] 
      |                                       
      |                                   
Flax.Build.exe  --[Generate]-> BuildScripts.csproj  Flax.vcxproj  FlaxEngine.csproj
                                           |
                                       [Combine]
                                           |
                                        Flax.sln



       ---[MSBUILD]---> FlaxEngine.exe 




```



#### 项目结构

1. 第一部分：项目生成工具（`Source\Tools\Flax.Build`），脚本会调用 MS Build 生成 `Flax.Build.exe`，然后该 exe 会运行执行 binding，使得 C# 和C++ 双向绑定。


2. 第二部分：实际项目



### 双向绑定

Flax 引擎通过自动生成的包装代码实现了 C++ 和 C# 之间的无缝双向绑定，该方法允许：
1. C++ 代码可以调用 C# 方法和事件
2. C# 代码可以访问 C++ 的属性、方法和事件
3. 支持引用参数的双向数据流
4. 接口的跨语言实现

整个过程是在构建时自动生成的，开发者无需手动编写互操作代码，主要通过 `BindingsGenerator`类来生成必要的包装代码。

Flax引擎使用两个主要的生成器类来创建绑定：
	- `BindingsGenerator.Cpp.cs` - 负责生成C++包装代码
	- `BindingsGenerator.CSharp.cs` - 负责生成C#包装代码

#### C++到C#的绑定

1. **事件绑定**
	- 为 C++ 事件创建 C# 包装器
	- 使用内部调用（internal calls）连接 C++ 和 C#
	- 创建专用的委托类型或使用 Action/Func

```csharp
// 生成事件代理定义
contents.Append(indent).Append("public delegate void ")
                       .Append(eventInfo.Name)
                       .Append("Delegate(");
// ...参数处理...
contents.Append(");").AppendLine().AppendLine();
```

2. **字段绑定**
	- 生成 getter/setter 函数
	- 通过这些函数实现对 C++ 字段的访问
```csharp
// 静态字段使用C++静态值，通过getter函数绑定访问
fieldInfo.Getter = new FunctionInfo
{
    Access = fieldInfo.Access,
    IsStatic = true,
    Parameters = new List<FunctionInfo.ParameterInfo>(),
    ReturnType = fieldInfo.Type,
    Name = fieldInfo.Name,
    UniqueName = "Get" + fieldInfo.Name,
};
```

#### C#到C++的绑定

1. **事件回调处理**：当C#事件被触发时，需要回调到C++代码：
```csharp
// C# 事件包装器绑定方法（绑定/解绑C#包装器到C++委托）
CppInternalCalls.Add(new KeyValuePair<string, string>(
										eventInfo.Name + "_Bind", 
					                    eventInfo.Name + "_ManagedBind"));
contents.AppendFormat("    DLLEXPORT static void {0}_ManagedBind(", eventInfo.Name);
```

2. **引用参数处理**：对于引用参数，系统需要确保数据可以双向流动：
```csharp
// 将值从托管转换回本机（可能在那里被修改）
paramType.IsRef = false;
var managedToNative = GenerateCppWrapperManagedToNative(
                            buildData, paramType, classInfo, 
                            out var managedType, out var apiType, null, out _);
```

#### 接口实现

支持创建包装类来实现接口的交互：
```csharp
// 创建包装接口实现，以便在C#或VS中继承时调用脚本
contents.AppendFormat("class {0}Wrapper : public ", interfaceTypeNameInternal)
        .Append(interfaceTypeNameNative)
        .AppendLine();
```

#### 项目组织结构

绑定代码通常放置在特定位置：
```csharp
// 将项目目标二进制模块绑定放在项目的Source文件夹中（Visual Studio以这种方式更好地处理C#源文件）
project.Path = Path.Combine(
                    project.WorkspaceRootPath, 
                    "Source", 
                    project.Name + '.' + dotNetProjectGenerator.ProjectFileExtension);
```

### 运行逻辑

1. **主循环**：`Source/Engine/Engine/Engine.cpp`，游戏引擎的主循环，负责控制引擎的整体流程，包含几个关键部分：
	- 平台事件更新
	- 游戏逻辑更新
	- 帧绘制
	- 物理模拟
	- CPU使用率管理和休眠逻辑
	- 应用暂停/恢复处理




## 分析

### 编程语言

Flax Engine 采用了混合语言架构：
- C++ ：引擎核心功能和底层系统使用 C++ 实现，负责性能关键的部分，如渲染、物理、内存管理等。
- C# ：用于高级游戏逻辑、编辑器功能和工具开发，使用 .NET 框架，提供更高级的编程接口和更快的开发速度。
- **两种语言之间通过绑定层进行通信**

这种架构结合了 C++ 的性能和 C# 的开发效率，是现代游戏引擎的常见设计模式。


### 编译过程

使用 C++ 和 C# 混合架构的游戏引擎，其编译过程涉及多个步骤和工具：

1. C++ 代码根据不同平台使用不同的编译器：
	- Windows ：Visual C++ 编译器（MSVC）
	- Linux ：GCC 或 Clang
	- macOS ：Clang
编译过程包括，编译源文件（CompileCppFiles 方法），链接目标文件（LinkFiles 方法）

2. C# 代码编译使用 .NET 编译器。
3. **绑定生成**：C++ 和 C# 之间的互操作通过绑定层实现：
	- 生成 C# 绑定代码
	- 编译绑定代码生成 `.dll` 文件


### 绑定生成

Flax Engine 使用了一套绑定系统，让 C# 代码能够与 C++ 代码进行交互。首先 Flax 使用**自动化工具**生成 C# 和 C++ 之间的绑定代码，在项目生成脚本中可以看到：
```bash
# 在 GenerateProjectFiles.sh 中
Binaries/Tools/Flax.Build -build -BuildBindingsOnly -arch=x64 -platform=Linux --buildTargets=FlaxEditor
```
`-BuildBindingsOnly` 参数表明这个命令专门用于生成绑定代码，而不是编译整个项目。

#### 绑定层的组成

1. C++ 侧导出机制 ：
	- 导出函数和类 ：C++ 代码使用特殊的宏和注解来标记需要导出到 C# 的函数、类和属性。
	- API 导出宏 ：`FLAXENGINE_API` 宏用于标记需要导出的类和函数。
	- 本地调用接口 ：C++ 代码实现了一系列可以被 P/Invoke 调用的函数。

2. C# 侧绑定机制 ：
	- 生成的绑定类：构建过程会生成 C# 包装类，这些类映射到 C++ 中的对应类。
	- `FlaxEngine.CSharp.dll`：这是主要的绑定库，包含了所有从 C++ 导出到 C# 的类和函数。
	- P/Invoke 调用：C# 代码通过 P/Invoke 机制调用 C++ 导出的函数。

#### 绑定工作流程

- 解析 C++ 代码：绑定生成器分析 C++ 代码，识别标记为导出的类和函数。
- 生成 C# 包装类：为每个导出的 C++ 类生成对应的 C# 类。
- 生成 P/Invoke 声明：为 C++ 导出函数生成 P/Invoke 声明。
- 编译绑定库：将生成的 C# 代码编译成 `FlaxEngine.CSharp.dll`。



## 使用

### Windows

1. 前置要求：通过 Visual Studio Installer 安装，
	- Visual Studio 2022 或更新的版本
	- Microsoft Visual C++ 2015 v140 toolset
	- .NET 8 or 9 SDK for **Windows x64**
2. 步骤一：运行 **GenerateProjectFiles.bat**
3. 步骤二：打开 `Flax.sln` ，配置解决方案为 **Editor.Development** ，平台为 **Win64**
4. 步骤三：设置 Flax (C++) 或者 FlaxEngine (C#) 为启动项目
5. 步骤四：编译项目
6. 步骤五：（Optionally）Debug 类型设置为 **Managed Only (.NET Core)** 可以只 debug C#，设置为 **Mixed (.NET Core)** 可以 debug C++ 和 C#
7. 步骤六：运行项目

### Mac
1. 前置要求：安装
	- XCode
	- [.NET 8](https://dotnet.microsoft.com/en-us/download/dotnet/8.0) or 9 SDK
	- [Vulkan](https://vulkan.lunarg.com/) SDK
2. 步骤一：运行 `GenerateProjectFiles.command`
3. 步骤二：用 XCode 或者 Visual Studio Code 打开 workspace 
4. 步骤三：编译，运行 (configuration `Editor.Mac.Development`)



