
使用 C++ 和 C# 混合开发可以结合两种语言的核心优势，能够在性能、开发效率和功能扩展性之间找到平衡。

常见于游戏引擎开发。

**主要优点**

1. **性能与生产力的平衡**
	- **C++ 负责底层高性能模块**：
		- **图形渲染**：直接调用 DirectX、Vulkan 或 OpenGL 等图形 API，最大化硬件性能。
		- **物理引擎**：实时碰撞检测、刚体动力学等计算密集型任务。
		- **内存管理**：通过手动内存控制优化资源分配（如对象池、自定义内存分配器）。
	- **C# 负责高层逻辑和工具链**：
		- **游戏逻辑**：通过脚本化的组件系统快速迭代玩法（如 Unity 的 MonoBehaviour）。
		- **编辑器开发**：利用 C# 的 WPF、WinForms 或跨平台框架（如 Avalonia）快速构建可视化工具。
		- **热重载**：运行时动态更新代码，无需重启引擎即可调试逻辑。

2. **跨平台兼容性**
	- **C++ 的跨平台基础**：
	    - 通过抽象层（如 SDL、GLFW）支持 Windows、Linux、macOS、主机和移动平台。
	    - 直接编译为原生代码，避免虚拟机开销。
	- **C# 的灵活部署**：
	    - 通过 .NET 6+ 或 Mono 实现跨平台脚本逻辑。
	    - Unity 等引擎利用 IL2CPP 将 C# 代码转换为 C++，优化性能并支持 iOS 等限制 JIT 的平台。

3. **生态系统与工具整合**
	- **C++ 生态优势**：
	    - 成熟的图形库（DirectX、Vulkan）、数学库（GLM、Eigen）和物理引擎（Bullet、PhysX）。
	    - 与硬件厂商（NVIDIA、AMD）的深度优化工具链（如 CUDA、OptiX）集成。
	- **C# 生态优势**：
	    - 强大的 IDE 支持（Visual Studio、Rider）提供代码分析、调试和重构。
	    - NuGet 包管理快速集成 JSON 解析、网络通信等通用功能。
	    - 与 Unity 编辑器、VSCode 等工具无缝协作，提升工作流效率。

4. **开发团队协作优化**
	- **分工明确**：
	    - **C++ 团队**：专注引擎核心（渲染管线、资源管理、多线程优化）。
	    - **C# 团队**：开发编辑器、脚本系统和游戏逻辑，快速响应设计需求。
	- **接口标准化**：
	    - 通过 C++/CLI、P/Invoke 或 SWIG 生成绑定，暴露底层功能（如 `RenderMesh()` 方法）。
	    - 使用中间件（如 Unity 的 Burst Compiler）实现高性能 C# 代码与 C++ 的无缝交互。

5. **实时调试与热更新**
	- **C# 的动态能力**：
	    - 运行时反射和动态加载（如 `Assembly.LoadFrom`）支持模组化架构。
	    - 游戏逻辑热更新（如修复 BUG 无需重新编译引擎）。
	- **C++ 的稳定性**：
	    - 核心引擎模块通过静态编译确保稳定性，避免动态语言的内存泄漏风险。

6. **面向数据的设计优化**
	- **C++ 处理数据密集型任务**：
	    - 使用 ECS（实体组件系统）架构优化缓存命中率（如 Unity DOTS）。
	    - SIMD 指令集（如 AVX2）加速矩阵运算、粒子系统更新。
	- **C# 管理逻辑数据**：
	    - 序列化游戏配置（JSON、XML）、管理关卡数据，利用 LINQ 快速查询。


## 构建流程工具链

在混合使用 C++ 和 C# 的游戏引擎开发中，构建工具链的配置需要兼顾两种语言的编译特性、跨平台需求和自动化流程。以下是具体的工具链配置方案：

### 1. **构建系统选择与整合**

- **C++ 构建系统**
	- **CMake**（推荐）：
	    - **跨平台支持**：生成 Visual Studio、Xcode、Makefile、Ninja 等工程文件。
	    - **依赖管理**：通过 `find_package` 集成第三方库（如 OpenGL、SDL2）。
	    - **混合构建示例**：
```c++
# 添加 C++ 主项目
add_subdirectory("EngineCore")  # C++ 引擎核心
add_subdirectory("NativePlugins")  # C++ 插件

# 调用 MSBuild 编译 C# 工具链
add_custom_command(
    TARGET EngineCore POST_BUILD
    COMMAND dotnet build "${CMAKE_SOURCE_DIR}/EditorTools/EditorTools.csproj --configuration Release"
)
```

- **C# 构建系统**
	- **.NET CLI**（跨平台）：
```bash
dotnet build EditorTools.csproj -c Release
dotnet publish ScriptRuntime.csproj -r win-x64 --self-contained
```

### 2. **自动化工具链配置**

- **CI/CD 流水线**
	- **GitHub Actions 示例**：
```yaml
name: Build Engine
on: [push]
jobs:
  build-windows:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install C++ Dependencies
        run: vcpkg install sdl2 glm --triplet x64-windows
      - name: Build C++ Engine
        run: cmake -B build -G "Visual Studio 17 2022" -A x64 && cmake --build build --config Release
      - name: Build C# Tools
        run: dotnet build EditorTools/EditorTools.csproj -c Release

  build-linux:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install C++ Dependencies
        run: sudo apt-get install libsdl2-dev libglm-dev
      - name: Build C++ Engine
        run: cmake -B build -G "Ninja" && cmake --build build --config Release
```


- **包管理工具**
	- **C++ 依赖**：
		- **vcpkg**（微软生态）：
		```c++
		vcpkg install physx bullet --triplet x64-windows
		```
		- **Conan**（跨平台）：
	  ```C++
		# conanfile.txt
		[requires]
		zlib/1.2.11
		glfw/3.3.8
		
		[generators]
		cmake_find_package
		```
	- **C# 依赖**：
		- **NuGet**：
	  ```xml
		<!-- .csproj 文件 -->
		<ItemGroup>
		  <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
		  <PackageReference Include="SharpDX" Version="4.3.0" />
		</ItemGroup>
		```


### 3. **跨语言交互配置**

- **P/Invoke 手动封装**（简单场景）：
```c#
// C# 调用 C++ 函数
[DllImport("EngineCore.dll", EntryPoint = "RenderFrame")]
public static extern void RenderFrame(IntPtr context);
```

- **自动绑定工具**（复杂项目）：
	- **C++/CLI**（仅 Windows）：
	```cpp
	// ManagedWrapper.cpp
	#pragma managed
	public ref class EngineBridge {
	public:
	    static void Initialize() { NativeEngine::Initialize(); }
	};
	```
	- **SWIG**（跨平台）：
	```
	// engine.i 接口文件
	%module Engine
	%{
	#include "engine.h"
	%}
	%include "engine.h"
	```
	 生成命令：
	```bash
	swig -csharp -c++ engine.i
	```

- **Unity 风格的 IL2CPP**：
    - 将 C# 代码转换为 C++ 源码后编译为原生二进制。

### 4. **构建优化策略**

**编译加速**

- **分布式构建**：
    - **Incredibuild**（商业工具）：加速 C++ 编译。
    - **ccache/sccache**（开源缓存）：
```bash
# CMake 配置
cmake -DCMAKE_C_COMPILER_LAUNCHER=ccache -DCMAKE_CXX_COMPILER_LAUNCHER=ccache ..
```

- **预编译头文件（PCH）**：
```cmake
# CMakeLists.txt
target_precompile_headers(EngineCore PRIVATE pch.h)
```

**依赖隔离**

- **C++ 模块化构建**：
```cmake
# 将引擎拆分为独立组件
add_library(Rendering STATIC rendering.cpp)
add_library(Physics STATIC physics.cpp)
target_link_libraries(EngineCore PRIVATE Rendering Physics)
```

- **C# 程序集分割**：
```xml
<!-- 将脚本运行时与编辑器工具分离 -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <AssemblyName>ScriptRuntime</AssemblyName>
    <EnableDynamicLoading>true</EnableDynamicLoading>
  </PropertyGroup>
</Project>
```

### 5. **跨平台构建配置**

**Windows**

- **Visual Studio 工具链**：
```bash
cmake -G "Visual Studio 17 2022" -A x64 -Thost=x64
```
- **MSVC 编译器优化**：
```cmake
target_compile_options(EngineCore PRIVATE /Ox /fp:fast /arch:AVX2)
```

**Linux/macOS**

- **Clang 工具链**：
```bash
cmake -G "Ninja" -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++
```
- **Mono 运行时集成**（C# 跨平台）：
```bash
# 安装 Mono 开发环境
sudo apt install mono-devel msbuild
```

**移动端（iOS/Android）**

- **Xcode 集成**：
```bash
cmake -G "Xcode" -DCMAKE_SYSTEM_NAME=iOS
```
- **Android NDK 配置**：
```cmake
cmake -DCMAKE_TOOLCHAIN_FILE=$NDK/build/cmake/android.toolchain.cmake -DANDROID_ABI=arm64-v8a
```

### 6. **调试与日志整合**

- **混合调试配置**（Visual Studio）：
    1. 在解决方案中同时加载 C++ 和 C# 项目。
    2. 右键解决方案 → 属性 → 启用“本机代码调试”（C# 项目设置）。
    3. 使用条件断点过滤跨语言调用栈。
- **统一日志系统**：
```cpp
// C++ 日志接口
extern "C" __declspec(dllexport) void LogMessage(const char* message) {
	std::cout << "[C++] " << message << std::endl;
}
```

```csharp
// C# 封装
public static class Logger {
	[DllImport("EngineCore.dll")]
	private static extern void LogMessage(string message);
	
	public static void Info(string msg) => LogMessage($"[INFO] {msg}");
}
```


### **典型目录结构**

```
Engine/
├── Binaries/           # 最终输出目录
├── Source/
│   ├── EngineCore/     # C++ 引擎核心 (CMake)
│   ├── EditorTools/    # C# 编辑器 (.NET SDK)
│   └── ScriptRuntime/  # C# 游戏脚本逻辑
├── ThirdParty/
│   ├── vcpkg/         # C++ 依赖管理
│   └── NuGet/         # C# 包缓存
├── BuildScripts/
│   ├── build.ps1      # Windows 构建脚本
│   └── build.sh       # Linux/macOS 构建脚本
└── CMakeLists.txt     # 根 CMake 配置
```



## 代码相互操作

C# 与 C++ 交互，总体来说可以有两种方法：
- **利用 C++/CLI 作为代理中间层**：实现起来比较简单直观，
	- 可以实现 C# 调用 C++ 所写的类；
	- .NET 支持
	- MONO 不支持
- **利用 P/Invoke 实现直接调用**：
	- 添加 DllImportAttribute 特性即可以导入 C/C++ 的函数
	- 不能简单的实现对 C++ 类的调用。

### 通过 C++/CLI 调用

C++/CLI（C++ for Common Language Infrastructure）是微软推出的一种编程语言扩展，旨在将C++ 与 .NET 框架（Common Language Infrastructure，CLI）相结合，使开发者能够在 C++ 中直接编写托管代码（Managed Code），并与 .NET 生态系统（如 C# 、VB.NET 等语言）无缝交互。


**主要原理**

- **编译器支持**：C++/CLI 编译器能够编译同时包含托管和非托管代码的源文件，这使得在同一个项目中可以混合使用 C++ 和 C# 或其他 .NET 语言。
- **托管和非托管代码的桥梁**：C++/CLI 提供了语法和关键字，允许你在同一个文件中编写托管代码和非托管代码。这些代码可以相互调用，使得 C# 和 C++ 之间的交互变得简单。
- **CLR（Common Language Runtime）集成**：C++/CLI 代码在编译时会生成对 CLR 的调用，因此它可以利用 CLR 提供的各种功能，例如垃圾回收、类型安全性和异常处理等。
- **托管代码的封装**：在 C++/CLI 中，你可以将非托管的 C++ 代码封装在托管的类中，通过公共接口暴露给其他 .NET 语言。这样，C++ 的功能可以被其他 .NET 语言轻松调用和使用。
- **数据类型转换**：C++/CLI 提供了一组转换操作符和工具，用于在托管代码和非托管代码之间进行数据类型的转换。这样，你可以在 C++/CLI 中轻松处理 C# 中的数据类型，反之亦然。
- **资源管理**：在 C++/CLI 中，你可以使用托管的资源管理功能（如 `gcnew` 创建托管对象和 `delete` 销毁对象），同时也可以手动管理非托管资源（如使用析构函数释放内存）。


C++/CLI 是连接本地 C++ 与 .NET 世界的桥梁，适用于需要同时利用两种环境优势的场景。尽管其复杂度较高且不如 C# 流行，但在需要高性能互操作的系统中（如游戏引擎、驱动开发），它仍是一个强大的工具。


### 通过 P/Invoke 调用

1. 从C++中导出动态链接库（ `*.dll` ）
	- 首先需要在C++代码中，声明方法之前加入 `extern "C" __declspec(dllexport)` 前缀，表示这个方法会被编译到 `*.dll` 中作为一个可供外部调用的方法。
	- 其中 `extern "C"` 是**当C和C++混合编写的时候使用**，用于告知编译器**以C语言规范编译**。
```C++
#pragma once

#ifdef __cplusplus
extern "C" {
#endif
	__declspec(dllexport) void re_init(GLFWwindow * wndPtr);
	// ...
#ifdef __cplusplus
}
#endif
```


2. 编译成 `*.dll` 文件
	- 在Visual Studio中，选择 `项目 -> 属性` ，然后在配置类型中设置为动态链接库(.dll)![](imgs/Pasted%20image%2020230921143540.png)
	- 同时对于不同的编译模式Debug、Release都需要分别配置，对应的编译平台 x86、x64 都需要设置完毕。



3. 将Release生成的 `*.dll` 文件导入C#中
	- 新建一个C#文件，导入 `System.Runtime.InteropServices` 库，然后新建一个类（可以是`abstract`）。
	- 使用C#特性 `[DLLImport"DLLFileName"] public extern static ...` 将 `*.dll` 中的方法作为外部静态成员方法导入。
```C#
using System.Runtime.InteropServices;

namespace Example {
    /// Example class handling the rendering for OpenGL.
    public static class ExampleScene {
    
        [DllImport("render-engine.dll")], CallingConvention = CallingConvention.Cdecl, CharSet = CharSet.None, ExactSpelling = false)]
		public static extern void re_init(Window* wndPtr);
		
		// ...
    }
}
```
这样就可以在C#脚本中调用，和一般的C#静态方法调用方式一样。

**注意**：
- `[DllImport("……")]` 中填写的是导入的 `*.dll` 文件名，标志将要导入的方法来自于哪个 `*.dll` 文件，函数名和参数数量需要对应。
- C#中并不能对应C++中所有参数类型，尤其是自定义结构体、类型等。


4. C#与C++类型对应
复杂的类型或者自定义结构体就需要在C#端重写一遍。下面的例子包含了几种我遇到的典型的参数类型：指针、引用、数组、结构体、`void*`。

如对于C++代码：
```c++
#define API extern "C" __declspec(dllexport)

struct Struct_A
{
    float fArray4[4];       // 结构体中包含数组
    float fArray8[8];   
    int iCount;
};

void API Function(
    uint32* vUInt32         // 指针
    void* pVoidPtr,         // void*
    &Struct_A sA,           // 自定义类型的引用
);
```

转化成C#的导入和使用代码是：
```csharp
// 导入
static class DllImporter{
    // 定义 struct
    // 规定 struct 成员布局
    [StructLayout(LayoutKind.Sequencial)]
    public struct Struct_A{
        // 规定非托管数据的大小，SizeCount为元素个数
        [MarshalAs(UnmanagedType.ByValArray, SizeCount = 4)]
        public float[] fArray4;
        [MarshalAs(UnmanagedType.ByValArray, SizeCount = 8)]
        public float[] fArray8;
        public int iCount;
    }
    // 引入dll方法
    [DLLImport("dllexample")]
    public static void Function(
        ref UInt32 vUInt32,
        IntPrt pVoidPtr,
        ref Struct_A sA
    )
}
​
// 使用
static class DllExample{
    static void Main(){
        // ...
        // init local array
        float[] localArray = new float[100];
        // calculate array's Size by Byte
        int sizeCount = Marshal.SizeOf(typeof(float)) * localArray.Length;
        // declare a IntPtr, and allocate heap space for it
        IntPtr pIntPtr = Marshal.AllocHGlobal(sizeCount);
        // copy datas to the address that IntPtr pointed
        Marshal.Copy(localArray, 0, pIntPtr, localArray.Length);
        // ...
        Function(ref vUInt32, pIntPtr, ref sA);
        // ...
        // free IntPtr's memory
        Marshal.FreeHGlobal(pIntPtr);
    }
}
```

