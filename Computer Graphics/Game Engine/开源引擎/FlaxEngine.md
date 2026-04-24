FlaxEngine 是一个开源的现代 3D 游戏引擎，使用 C++ 和 C# 双语言开发，完整覆盖从底层渲染到上层游戏逻辑的所有系统，支持 Windows / Linux / Mac / Android / iOS / Web 多平台。它提供了一套现代化的工具和功能，支持从小型独立游戏到大型商业项目的开发。

## 目录结构

```
FlaxEngine/
├── Source/
│   ├── Engine/          # 核心引擎（C++ + C# 双端）
│   ├── Editor/          # 编辑器（C++ + C# 双端）
│   ├── Platforms/       # 平台适配层（含 .NET runtime 依赖）
│   ├── Shaders/         # 着色器源码
│   ├── ThirdParty/      # 外部依赖
│   └── Tools/
│       └── Flax.Build/  # 自研构建系统（纯 C# 实现）
├── Content/             # 引擎/编辑器资源
├── Binaries/            # 编译产物
└── Development/Scripts/ # 平台专属生成脚本
```


## 架构

### C++ 与 C# 职责划分

| 子系统       | C++ | C#  | 说明                           |
| --------- | --- | --- | ---------------------------- |
| 渲染/GPU    | ✓   |     | GraphicsDevice 抽象、RenderTask |
| 物理        | ✓   |     | 刚体模拟                         |
| 音频        | ✓   |     | 音频播放与混音                      |
| 输入        | ✓   |     | 平台输入采集                       |
| 平台/OS     | ✓   |     | 窗口、文件 I/O、线程                 |
| 内容系统      | ✓   |     | 资源加载/Cook/序列化                |
| 场景图       | ✓   |     | Actor 管理                     |
| 骨骼动画      | ✓   |     | Skeletal 动画、曲线求值             |
| 粒子/导航/CSG | ✓   |     | 实时系统                         |
| 编辑器核心     | ✓   | ✓   | C++ 底层 + C# UI 与工具           |
| **游戏脚本**  |     | ✓   | 用户继承 `Script` 类写游戏逻辑         |
| **构建系统**  |     | ✓   | Flax.Build 全部由 C# 实现         |

**核心原则**：C++ 负责所有实时性要求高的系统；C# 负责游戏脚本逻辑和编辑器 UI。



### C++ 与 C# 互调机制

#### 第一层：构建时绑定代码生成（BindingsGenerator）

Flax 引擎通过**自动生成**的包装代码实现了 C++ 和 C# 之间的无缝双向绑定，该方法允许：
1. C++ 代码可以调用 C# 方法和事件
2. C# 代码可以访问 C++ 的属性、方法和事件
3. 支持引用参数的双向数据流
4. 接口的跨语言实现

整个过程是在构建时自动生成的，开发者无需手动编写互操作代码，主要通过 `BindingsGenerator`类来生成必要的包装代码。

核心源码位于 `Source/Tools/Flax.Build/Bindings/`，是整个 Interop 机制的核心。

**工作流程**：

```
C++ 头文件 (.h)
   ↓  标注 API_CLASS / API_FUNCTION / API_STRUCT 宏
BindingsGenerator 扫描 & 解析
   ↓
生成 C++ 侧 wrapper（参数 marshalling + 调用）
生成 C# 侧 P/Invoke 声明 / internal call stub
   ↓
FlaxEngine.Gen.h / .Gen.cpp / .Gen.cs（自动生成，禁止手改）
```

**核心源码**：
- `BindingsGenerator.Cpp.cs`：生成 C++ 胶水代码
- `BindingsGenerator.CSharp.cs`：生成 C# 调用声明


**绑定流程**：
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
2. **事件回调处理**：当 C# 事件被触发时，需要回调到 C++ 代码：
```csharp
// C# 事件包装器绑定方法（绑定/解绑C#包装器到C++委托）
CppInternalCalls.Add(new KeyValuePair<string, string>(
										eventInfo.Name + "_Bind", 
					                    eventInfo.Name + "_ManagedBind"));
contents.AppendFormat("    DLLEXPORT static void {0}_ManagedBind(", eventInfo.Name);
```

3. **字段绑定**
	- 生成 getter/setter 函数
	- 通过这些函数实现对 C++ 字段的访问
```csharp
// 静态字段使用 C++ 静态值，通过 getter 函数绑定访问
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

4. **引用参数处理**：对于引用参数，系统需要确保数据可以双向流动：
```csharp
// 将值从托管转换回本机（可能在那里被修改）
paramType.IsRef = false;
var managedToNative = GenerateCppWrapperManagedToNative(
                            buildData, paramType, classInfo, 
                            out var managedType, out var apiType, null, out _);
```

5. 支持创建包装类来实现接口的交互：
```csharp
// 创建包装接口实现，以便在C#或VS中继承时调用脚本
contents.AppendFormat("class {0}Wrapper : public ", interfaceTypeNameInternal)
        .Append(interfaceTypeNameNative)
        .AppendLine();
```

6. 绑定代码通常放置在特定位置：
```csharp
// 将项目目标二进制模块绑定放在项目的Source文件夹中（Visual Studio以这种方式更好地处理C#源文件）
project.Path = Path.Combine(
                    project.WorkspaceRootPath, 
                    "Source", 
                    project.Name + '.' + dotNetProjectGenerator.ProjectFileExtension);
```


#### 第二层：运行时托管 CLR 抽象层（ManagedCLR）

核心源码位于 `Source/Engine/Scripting/ManagedCLR/`，将引擎与具体 .NET 运行时解耦：

```
MCore.h        — CLR 核心服务
MClass.h       — 托管类型信息
MMethod.h      — 托管方法调用
MUtils.h       — 类型 marshalling（Box/Unbox、字符串转换）
```

运行时实现由 `Source/Engine/Scripting/Runtime/` 提供：

|文件|对应运行时|编译开关|
|---|---|---|
|`DotNet.cpp`|CoreCLR / .NET 5+|`USE_NETCORE`|
|`Mono.cpp`|Mono|`USE_MONO`|
|`None.cpp`|无脚本|—|






#### 第三层：对象桥接（ScriptingObject）

在 `Source/Engine/Scripting/ScriptingObject.h` 中所有要暴露给 C# 的 C++ 类都继承自 `ScriptingObject`：

```cpp
class ScriptingObject : public Object {
    MGCHandle _gcHandle;       // 指向对应的托管对象，防止 GC 回收
    ScriptingTypeHandle _type; // 类型元信息
    MObject* GetOrCreateManagedInstance(); // 按需创建 C# 侧实例
};
```

C++ 与 C# 对象通过 GC Handle 保持双向引用，生命周期互相感知。

#### Internal Calls 的双模式实现

在 `Source/Engine/Scripting/Internal/InternalCalls.h`：

```cpp
#if USE_NETCORE
// .NET Core：直接导出 C 函数
#define DEFINE_INTERNAL_CALL(returnType) extern "C" DLLEXPORT returnType

#else  // Mono
// Mono：通过 mono_add_internal_call 注册
#define ADD_INTERNAL_CALL(fullName, method) \
    mono_add_internal_call(fullName, (const void*)method)
#endif
```

.NET Core 侧对应的 C# 桥接在 `Source/Engine/Engine/NativeInterop.cs`，使用 `[UnmanagedCallersOnly]` + `ManagedHandle` 实现零拷贝互调。

### 完整互调示例

1. **C# 调用 C++ 渲染函数的路径**：
```
C# 代码调用 Actor.SetPosition(...)
    ↓
BindingsGenerator 生成的 C# P/Invoke stub
    ↓
C++ wrapper（参数从 MObject* 转换为 Vector3）
    ↓
实际 C++ Actor::SetPosition 实现
    ↓
引擎内部通知 Transform 更新
```

2. **C++ 回调 C# 脚本的路径**：
```
引擎主循环 Update()
    ↓
ScriptingObject 通过 MGCHandle 找到 C# Script 实例
    ↓
MMethod::Invoke 调用 C# 的 OnUpdate()
    ↓
用户 C# 游戏逻辑执行
```


## 编译机制

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


### 核心工具：Flax.Build

整个编译系统的入口是自研构建工具 `Flax.Build`（纯 C# 实现，位于 `Source/Tools/Flax.Build/`，它的职责相当于 CMake + MSBuild + 代码生成器的合体。

#### `Flax.Build` 的启动流程

`Program.cs`：

```
Main()
  1. 解析命令行参数 → Configuration 对象
  2. 找到工作目录下的 *.flaxproj 文件并加载（项目元数据）
  3. GenerateRulesAssembly()   ← 编译所有 *.Build.cs 规则文件
  4. 执行指定操作：
       --build         → 编译目标
       --generateprojectfiles → 生成 .sln / .vcxproj
       --clean         → 清理产物
       --deploy        → 打包部署
```

### 模块规则系统（`*.Build.cs`）

#### Rules 编译（动态编译规则）

`Builder.Rules.cs` — **Flax.Build 首先把项目下所有的 `*.Build.cs` 文件作为 C# 源码在内存里动态编译成一个 Assembly**，然后用反射实例化其中的 `Target` / `Module` / `Plugin` 子类：

```
Source/ 下所有 *.Build.cs
         ↓ Roslyn 动态编译
     RulesAssembly (in-memory)
         ↓ 反射
[Target]    [Module]    [Plugin]
FlaxEditor   Audio        ...
FlaxGame     Renderer
             Physics
             Content
             ...
```

#### Module 定义示例

每个引擎子系统都有对应的 `*.Build.cs`，例如 `Source/Engine/AI/AI.Build.cs`：

```csharp
public class AI : EngineModule
{
    public override void Setup(BuildOptions options)
    {
        base.Setup(options);
        options.PublicDependencies.Add("Navigation"); // 声明依赖
    }
}
```


### 完整编译流程（引擎本体）

```
┌─────────────────────────────────────────────────────────────────┐
│                        Flax.Build 流程                          │
│                                                                 │
│  *.Build.cs → [动态编译] → Target/Module 对象图                  │
│                                    ↓                             │
│              CollectModules()  依赖图拓扑排序                     │
│                                    ↓                             │
│  ┌─── Step A: 绑定代码生成 (BindingsGenerator) ───────────────┐  │
│  │  扫描 C++ 头文件中的 API_CLASS / API_FUNCTION / API_FIELD  │  │
│  │  → 生成 C++ wrapper（参数 marshal + 调用原始函数）          │  │
│  │  → 生成 C# P/Invoke / internal call 声明                  │  │
│  │  输出：Cache/Intermediate/.../xxxx.Bindings.Gen.cpp/.cs   │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                    ↓                             │
│  ┌─── Step B: C++ 编译（NativeCpp）────────────────────────┐    │
│  │  Platform → Toolchain（MSVC / Clang）                   │    │
│  │  CompileEnvironment（defines / includes / flags）        │    │
│  │  TaskGraph → 并行编译 .cpp → .obj                        │    │
│  │  LinkEnvironment → 链接 .obj → .exe / .dll               │    │
│  └───────────────────────────────────────────────────────────┘   │
│                                    ↓                             │
│  ┌─── Step C: C# 编译（DotNet）────────────────────────────┐    │
│  │  收集所有模块的 .cs 文件                                  │    │
│  │  调用 dotnet build / csc                                 │    │
│  │  输出：FlaxEngine.CSharp.dll                             │    │
│  └───────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```


### 绑定代码生成（最关键的步骤）

#### 触发时机

在 C++ 编译之前，BindingsGenerator 会对每个模块的头文件进行扫描。

#### 宏标注 → 代码生成

开发者在 C++ 头文件中用宏标注要暴露的 API：

```cpp
// Actor.h
API_CLASS()
class FLAXENGINE_API Actor : public ScriptingObject
{
    API_FUNCTION()
    void SetPosition(API_PARAM(Ref) const Vector3& pos);

    API_PROPERTY()
    Transform GetTransform() const;
};
```

`BindingsGenerator.Parsing.cs` 解析头文件，`BindingsGenerator.Cpp.cs`（3488行）和 `BindingsGenerator.CSharp.cs`（2450行）分别生成两侧代码：

**生成的 C++ 端（胶水函数）：**

```cpp
// Auto-generated
extern "C" DLLEXPORT void Actor_SetPosition(Actor* obj, Vector3* pos) {
    INTERNAL_CALL_CHECK(obj);
    obj->SetPosition(*pos);
}
```

**生成的 C# 端（P/Invoke 声明）：**

```csharp
// Auto-generated
[DllImport("FlaxEngine", EntryPoint = "Actor_SetPosition")]
internal static extern void Internal_SetPosition(IntPtr obj, ref Vector3 pos);

public void SetPosition(Vector3 pos) => Internal_SetPosition(unmanagedPtr, ref pos);
```

#### VTable 注入（虚函数重写）

`InternalCalls.h:13-31` 中的 `VTableFunctionInjector`（仅 Clang）：当 C# 脚本重写了 C++ 虚函数时，BindingsGenerator 会生成代码在运行时直接修改 C++ 的 vtable，把虚函数指针替换成 C# 端的回调，析构时自动还原。


### Internal Calls 的双模式（InternalCalls.h 详解）

你打开的这个文件定义了整套互调的核心宏，根据运行时分两套路：

```cpp
#if USE_NETCORE                              // .NET Core / .NET 5+
  #define ADD_INTERNAL_CALL(name, method)    // 空！.NET Core 不需要注册
  #define DEFINE_INTERNAL_CALL(ret) extern "C" DLLEXPORT ret
  // C# 侧用 [DllImport] 直接找导出符号

#else                                        // Mono
  #define ADD_INTERNAL_CALL(name, method) \
      mono_add_internal_call(name, (const void*)method)  // 向 Mono 注册
  #define DEFINE_INTERNAL_CALL(ret) static ret
  // C# 侧用 [MethodImpl(MethodImplOptions.InternalCall)]
#endif
```

|模式|C++ 侧|C# 侧|适用平台|
|---|---|---|---|
|.NET Core|`DLLEXPORT` 导出 C 函数|`[DllImport]` P/Invoke|PC/主机|
|Mono|`static` 函数 + `mono_add_internal_call` 注册|`[InternalCall]`|iOS/Android/AOT|


### 游戏项目的编译流程

游戏项目和引擎共用 `Flax.Build`，但编译的是不同的 Target。

#### 游戏项目结构

```
MyGame/
├── MyGame.flaxproj        ← 项目描述（指定 GameTarget/EditorTarget）
└── Source/
    └── MyGame/
        ├── MyGame.Build.cs   ← 声明游戏模块 + 依赖引擎模块
        ├── PlayerScript.cpp
        ├── PlayerScript.h    ← API_CLASS() 标注后可暴露给 C#
        └── PlayerScript.cs   ← 纯 C# 游戏脚本（继承 Script）
```

#### 游戏项目 Build.cs 示例

```csharp
public class MyGame : GameModule  // 继承 GameModule
{
    public override void Setup(BuildOptions options)
    {
        base.Setup(options);
        // 自动依赖 FlaxEngine 核心模块
        // 可以添加额外依赖：
        options.PublicDependencies.Add("Audio");
    }
}
```

#### 游戏项目完整编译链

```
MyGame.flaxproj
      ↓
Flax.Build 加载 → GameProjectEditorTarget (编辑器内)
                   GameProjectTarget (打包发布)
      ↓
1. BindingsGenerator 扫描游戏 C++ 头文件
   → 生成游戏侧绑定代码（可选，游戏纯 C# 则跳过）

2. C++ 编译（如果有游戏 C++ 代码）
   → 链接引擎 FlaxEngine.dll（已编译好的引擎库）
   → 输出 MyGame.dll（原生游戏库）

3. C# 编译
   → 引用 FlaxEngine.CSharp.dll（引擎 C# API）
   → 编译所有 .cs 脚本
   → 输出 MyGame.CSharp.dll
```

#### 编辑器内的热重载

引擎启动后，`Scripting.h` 中的 `Scripting::Load()` 负责运行时加载程序集：

```
编辑器检测到 .cs 文件变化
      ↓
触发 Scripting::Reload()  [Scripting.h:107]
      ↓
ScriptsReloading 事件 → 保存场景对象状态
      ↓
卸载旧 MyGame.CSharp.dll（从 CLR 卸载 Assembly）
      ↓
Flax.Build 重新编译 C# 程序集（后台进程）
      ↓
LoadBinaryModules() 加载新 dll
      ↓
ScriptsReloaded 事件 → 还原场景对象状态
```


### 发布（打包）编译流程

```
Editor → Project Settings → Build
      ↓
Cook Pipeline
  1. Flax.Build 以 Release 配置重新编译
     - C++ → 关闭 DEBUG 宏，启用优化，去除 Editor 模块
     - C# → IL 编译，AOT（iOS/Android 下 Mono AOT）
  2. 资源 Cook（压缩 / 格式转换）
  3. 打包：
     - Windows：FlaxGame.exe + *.dll
     - Android：.apk（含 Mono AOT .so）
     - iOS：.ipa（含 AOT 静态库）
     - Web：.wasm + .js
```


### 完整流程一图总览

```
源码
 ├── C++ (.cpp/.h)          C# (.cs)
 │   API_CLASS 标注          Script 子类
 │        │                      │
 │   BindingsGenerator           │
 │   ├── 解析头文件               │
 │   ├── 生成 C++ wrapper        │
 │   └── 生成 C# stub            │
 │        │                      │
 │   C++ 编译                C# 编译
 │  (MSVC/Clang)          (dotnet/csc)
 │        │                      │
 │   .obj → 链接               编译
 │        │                      │
 │  FlaxEngine.dll      FlaxEngine.CSharp.dll
 │  MyGame.dll          MyGame.CSharp.dll
 │        │                      │
 └────────┴──────────────────────┘
                  │
              运行时加载
      CoreCLR hostfxr / Mono
                  │
      Scripting::Load() 加载 BinaryModule
                  │
      C++ ↔ C# 通过 Internal Call / P/Invoke 互调
```

核心设计：**编译时代码生成**消除了手写绑定的负担，**ManagedCLR 抽象层**让引擎在 .NET Core 与 Mono 之间自由切换，**任务图（TaskGraph）并行执行**保证大型项目编译速度。




## 运行机制

### 一、引擎启动链

```
WinMain / main()                    [Source/Engine/Main/Windows/main.cpp]
   ↓
Platform::PreInit()                 GPU 驱动偏好（NVIDIA/AMD）
   ↓
Engine::Main(cmdLine)               [Engine.cpp:239]
   ↓
Engine::OnInit()                    [Engine.cpp:77]
   ├── Platform::Init()             窗口系统、线程初始化
   ├── EngineService::OnInit()      所有服务按优先级顺序初始化
   │     ├── ScriptingService       → MCore::LoadEngine()  启动 CLR
   │     ├── GraphicsDevice         → GPUDevice::Instance 创建
   │     ├── PhysicsService         → PhysX/JOLT 初始化
   │     └── Editor (USE_EDITOR)    → ManagedEditor::Init()
   └── Application::BeforeRun()
   ↓
while (!ShouldExit()) { Engine::OnLoop(); }   ← 主循环
```

### 二、主循环：三条独立时间轴

`Engine.cpp` 的 `OnLoop()` 是引擎心跳，是**三条相互独立的时间轴**：

```cpp
void Engine::OnLoop()
{
    // 1. 节流睡眠：如果运行过快，sleep(1ms) 节省 CPU
    if (Time::UpdateFPS > 0 || !Platform::GetHasFocus())
    {
        if (timeToTick > 0.002) Platform::Sleep(1);
    }

    const double time = Platform::GetTimeSeconds();

    Platform::Tick();                         // OS 消息泵 / 输入采集

    if (Time::OnBeginUpdate(time)) {          // 可变频率（默认不限）
        OnUpdate();                           // 游戏逻辑
        OnLateUpdate();
        Time::OnEndUpdate();
    }

    if (Time::OnBeginPhysics(time)) {         // 固定频率（默认 60Hz）
        OnFixedUpdate();
        OnLateFixedUpdate();
        Time::OnEndPhysics();
    }

    if (Time::OnBeginDraw(time)) {            // 可变频率（DrawFPS，默认不限）
        OnDraw();
        Time::OnEndDraw();
    }
}
```

三条时间轴由 `Time` 的三个 `TickData` 对象（`Update`、`Physics`、`Draw`）分别管理，互不阻塞。一帧内物理可能跑 0 次也可能跑多次。


### 三、Update 链：C++ → C# 脚本

```
Engine::OnUpdate()                  [Engine.cpp:342]
  ├── MainThreadTask::RunAll()       执行主线程延迟任务
  ├── Engine::Update 事件            C++ delegate，通知各订阅者
  ├── EngineService::OnUpdate()      所有服务按顺序更新
  │     ├── InputService             处理输入状态
  │     ├── ScriptingService ─────── ← C# 脚本更新的入口
  │     │     └── Scripting::Update  C++ delegate 触发 C# 侧注册的 handler
  │     │           ↓
  │     │       SceneTicking::UpdateTickData::TickScripts()
  │     │           ↓
  │     │       for (Script* s : scripts) s->OnUpdate()  ← 虚函数
  │     │           ↓
  │     │       [BindingsGenerator 生成的 vtable wrapper]
  │     │           ↓
  │     │       C# Script.OnUpdate() 用户代码
  │     └── AnimationService、ContentService、UIService ...
  └── UpdateGraph->Execute()         异步任务图（JobSystem 多线程）
```

**关键细节**：`ScriptingService` 通过 C++ `Delegate<>` 模式触发，`/Source/Engine/Scripting/Scripting.h` 中定义了 `static Delegate<> Update`，对应 C# 侧注册的场景 tick 回调。

每个 `Scene` 持有一个 `SceneTicking` 实例，负责维护当前场景中所有已启用 `Script` 的有序列表。


### 四、Physics 链

```
Engine::OnFixedUpdate()             [Engine.cpp:306]  固定步长（60Hz）
  ├── Physics::FlushRequests()      应用本帧所有待处理的物理状态修改
  ├── FixedUpdate 事件
  ├── EngineService::OnFixedUpdate()
  │     └── ScriptingService → Script.OnFixedUpdate()  C# 物理帧脚本
  └── Physics::Simulate(dt)         ← 提交模拟，PhysX/JOLT 开始运算
                                      此后禁止修改物理对象状态！

Engine::OnLateFixedUpdate()         [Engine.cpp:328]
  ├── Physics::CollectResults()     取回模拟结果，触发碰撞事件
  ├── LateFixedUpdate 事件
  └── Script.OnLateFixedUpdate()    C# 侧响应碰撞结果
```

`Physics::Simulate()` 之后到 `CollectResults()` 之间，物理引擎在后台并行运算，引擎可以同期做渲染（auto-simulation 模式，注释见 `Source/Engine/Engine/Engine.cpp`。


### 五、渲染链：从 C++ 调度到 GPU 指令

#### 层级一：`Engine::OnDraw` → `GPUDevice::Draw`

```
Engine::OnDraw()                    [Engine.cpp:377]
  ├── FrameCount++
  ├── GPUDevice->Locker.Lock()      GPU 排他锁
  ├── GPUDevice::Draw()             [GPUDevice.cpp:781]
  │     ├── context->FrameBegin()   GPU 上下文帧开始
  │     ├── Render2D::BeginFrame()  2D 渲染层初始化
  │     ├── Engine::Draw 事件        自定义渲染订阅
  │     ├── EngineService::OnDraw() 服务级渲染（如编辑器 Gizmo）
  │     └── RenderTask::DrawAll()   ← 核心：执行所有渲染任务
  └── GPUDevice->Locker.Unlock()
```

#### 层级二：RenderTask 系统

`Source/Engine/Graphics/RenderTask.cpp` 中 RenderTask 是连接 C# UI 与 C++ 渲染器的桥梁：

```
RenderTask::DrawAll()
  ├── 按 Order 排序所有注册的 RenderTask
  └── for each task:
        if task.CanDraw():
          task.OnDraw()
            ├── OnBegin(ctx)    → Begin 事件（C# 可订阅）+ SwapChain::Begin
            ├── OnRender(ctx)   → Render 事件（C# 可订阅）
            │     └── if SwapChain:
            │           Render2D::Begin(backbuffer)
            │           SwapChain->GetWindow()->OnDraw()  ← GUI 绘制
            │           Render2D::End()
            └── OnEnd(ctx)      → SwapChain::End + End 事件
```

#### 层级三：SceneRenderTask → Renderer

`Source/Engine/Graphics/RenderTask.h` 中 `SceneRenderTask` 继承 `RenderTask`，是场景渲染的具体实现：

```
SceneRenderTask::OnRender(ctx)
  ├── 收集可见 Actor（视锥裁剪）
  ├── Renderer::Render(renderContext)
  │     ├── GBuffer Pass         几何写入（位置/法线/Albedo）
  │     ├── Shadow Pass          阴影贴图生成
  │     ├── Lighting Pass        直接光/IBL
  │     ├── Transparency Pass    半透明物体
  │     └── PostProcess Pass     TAA / Bloom / Tone Mapping
  └── 输出到 RenderBuffers（GPU 纹理）
```

### 六、编辑器 C# UI 的接入点

编辑器 UI 通过 **C# 事件订阅 + RenderTask** 挂载到渲染管线：

#### 启动时初始化

```
ManagedEditor::Init()               [ManagedEditor.cpp:185]
  ├── 通过反射找到 C# FlaxEditor.Editor 类
  ├── 创建 C# 侧托管实例（ScriptingObject::GetOrCreateManagedInstance）
  └── 调用 C# Editor.Init(flags, sceneId)
        ↓
        C# FlaxEditor 初始化
        ├── 创建主窗口（FlaxEditor.Windows.MainEditorWindow）
        ├── 创建 DockPanel 布局
        └── 创建各编辑器窗口（SceneEditorWindow、AssetBrowser 等）
```

#### 每帧 C# Editor 更新

```
EngineService::OnUpdate()
  └── EditorService::Update()
        └── ManagedEditor::Update()           [ManagedEditor.cpp:278]
              └── UpdateMethod->Invoke()      ← 通过反射调用 C# Editor.Update()
                    ↓
                    C# Editor.Update()
                    ├── 处理快捷键
                    ├── 更新工具栏状态
                    └── 刷新各 EditorWindow
```

#### EditorViewport 的渲染路径

```
C# EditorViewport（继承 RenderOutputControl）
  ├── 创建 SceneRenderTask 实例
  │     └── C++ 侧自动注册到 RenderTask::Tasks 列表
  ├── 设置 task.Camera = EditorCamera
  └── 订阅 task.Render 事件 → 用于叠加 Gizmo / Grid

每帧渲染时：
  GPUDevice::Draw()
    └── RenderTask::DrawAll()
          └── 遍历到该 SceneRenderTask
                └── 渲染场景 → 输出到 RenderOutputControl 的 GPU 纹理
                      ↓
                RenderTask::OnRender() 中的 Render 事件
                  └── C# 回调：绘制 Gizmo、坐标轴、选择框等 2D 叠加层
```

#### GUI 最终合成

```
主窗口 RenderTask（绑定 SwapChain）
  └── RenderTask::OnRender()            [RenderTask.cpp:104]
        ├── Render 事件（自定义内容）
        └── Render2D::Begin(backbuffer)
              SwapChain->GetWindow()->OnDraw()   ← 递归绘制整个 GUI 树
                ├── DockPanel.Draw()
                │     ├── SceneEditorWindow.Draw()
                │     │     └── EditorViewport（把之前渲染的纹理 blit 到此处）
                │     ├── AssetBrowser.Draw()
                │     └── InspectorWindow.Draw()
              Render2D::End()
```

### 七、完整一帧时序图

```
┌─────────────────────────────────── 一帧 ──────────────────────────────────┐
│                                                                            │
│  Platform::Tick()                                                          │
│  ├── OS 消息泵（窗口消息、输入事件）                                          │
│  └── 输入状态更新                                                            │
│                                                                            │
│  ── Update（可变频率）──────────────────────────────────────────────────    │
│  Engine::OnUpdate()                                                        │
│  ├── InputService        Raw 输入 → 虚拟轴映射                              │
│  ├── ScriptingService    C# Script.OnUpdate()（所有启用脚本）               │
│  ├── AnimationService    骨骼动画求值（可并行 JobSystem）                     │
│  ├── UIService           UI 事件分发                                        │
│  └── UpdateGraph::Execute()   异步任务并行执行                               │
│                                                                            │
│  Engine::OnLateUpdate()                                                    │
│  └── Script.OnLateUpdate()   所有 Update 完成后的补偿逻辑                   │
│                                                                            │
│  ── Physics（固定 60Hz）────────────────────────────────────────────────   │
│  Engine::OnFixedUpdate()                                                   │
│  ├── Physics::FlushRequests()  应用 SetVelocity / AddForce 等              │
│  ├── Script.OnFixedUpdate()                                                │
│  └── Physics::Simulate(dt)    ← PhysX/JOLT 开始并行模拟                    │
│                                                                            │
│  Engine::OnLateFixedUpdate()                                               │
│  ├── Physics::CollectResults() 接收结果 → OnCollisionEnter/Exit 等事件      │
│  └── Script.OnLateFixedUpdate()                                            │
│                                                                            │
│  ── Draw（可变频率）────────────────────────────────────────────────────   │
│  Engine::OnDraw()                                                          │
│  └── GPUDevice::Draw()                                                     │
│       ├── context->FrameBegin()    GPU 指令缓冲开始                         │
│       ├── Engine::Draw 事件        自定义 GPU 操作                           │
│       ├── EngineService::OnDraw()  编辑器 Gizmo GPU 命令                    │
│       └── RenderTask::DrawAll()                                            │
│            ├── [SceneRenderTask] 场景渲染                                   │
│            │    ├── GBuffer / Shadow / Light / PostFX Passes               │
│            │    └── 输出到离屏纹理                                           │
│            └── [MainWindowRenderTask] 窗口合成                              │
│                 ├── Render2D::Begin()                                      │
│                 ├── C# GUI 树递归绘制（Dock/Panel/Viewport/Widget）          │
│                 │    └── EditorViewport 将场景纹理 blit 至此                │
│                 └── SwapChain::Present()  ← 呈现到屏幕                      │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

### 八、核心设计模式总结

| 机制                             | 作用                              | 体现位置                                     |
| ------------------------------ | ------------------------------- | ---------------------------------------- |
| **EngineService**              | 各子系统按优先级有序初始化/更新，类似 ECS System  | Engine.cpp OnUpdate 链                    |
| **Delegate / Action**          | C++ 侧的事件系统，C# 可订阅               | `Scripting::Update`、`RenderTask::Render` |
| **RenderTask**                 | 解耦渲染目标与渲染内容，C# 创建任务自动注册到 C++ 列表 | RenderTask.cpp:39                        |
| **三时间轴**                       | Update/Physics/Draw 独立运行，避免相互阻塞 | Engine::OnLoop                           |
| **SceneRenderTask::Render 事件** | C# 编辑器在此叠加 Gizmo，无需侵入 C++ 渲染器   | EditorViewport.cs                        |
| **JobSystem/UpdateGraph**      | 动画、物理后处理等耗时任务多线程并行              | UpdateGraph->Execute()                   |

核心思路：**C++ 构造稳固的帧调度骨架**（三时间轴 + EngineService + RenderTask 列表），**C# 通过事件订阅和 RenderTask 注册挂载进来**，两者之间没有硬编码的调用关系，而是通过 Delegate/事件完全解耦。






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



