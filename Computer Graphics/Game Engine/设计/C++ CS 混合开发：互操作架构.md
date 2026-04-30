
## 架构概览

```
   C# 用户脚本                [LibraryImport]              C++ 运行时
  ┌───────────────────┐      ┌───────────────┐          ┌─────────────────────────┐
  │ Script.OnUpdate() │ ──→  │ InternalCalls │ ──→      │ Script_OnUpdate()       │
  │ (覆盖虚方法)       │      │ .OnUpdate()   │ P/Invoke │ (DEFINE_INTERNAL_CALL)  │
  └───────────────────┘      └───────────────┘          │ static_cast<Script*>    │
        ↑                                               │ → obj->OnUpdate()       │
        │                                               └────────┬────────────────┘
        │                                               ┌────────┴────────────────┐
  [UnmanagedCallersOnly]                                │ ScriptingEngine         │
  ManagedCallback_OnUpdate()  ← C++ 函数指针调用 ←       │ → 检查托管虚函数表        │
  → Object.FromUnmanagedPtr()                           │ → 调用 fnPtr 或 C++ impl │
  → 调用实际的 C# 重写                                   └─────────────────────────┘
```


互操作分为两个方向，共三层机制：

## 方向 1：C# 调用 C++（P/Invoke）

这是较简单的方向 —— C# 通过 `[LibraryImport]` 调用原生代码。

### 流程

1. **标注 C++ 头文件** —— 开发者使用 JzRE 的标注宏来标记 API 表面。例如，在 `Script.h` 中：
```cpp
API_CLASS()
class Script : public JzObject
{
    API_FUNCTION()
    virtual void OnUpdate(float deltaTime);
};
```

2. **`HeaderParser`**（`Source/Tools/JzRE.Build/Bindings/HeaderParser.cs`）：使用基于分词器的 C++ 解析器扫描这些标记，并构建一个包含 `ClassInfo`、`FunctionInfo`、`ParameterInfo` 等的 `ModuleInfo` 数据模型。

3. **`CppGenerator`** ：生成胶水代码，对于每个被标记的方法，它会写出一个 `extern "C"` 的桩函数，将 `void* __self` 参数转换为具体的 C++ 类：
```cpp
// Generated in JzRE.Runtime.Bindings.Gen.cpp
DEFINE_INTERNAL_CALL(void) Script_OnUpdate(void* __self, float deltaTime)
{
    INTERNAL_CALL_CHECK(__self);
    auto* _obj = static_cast<Script*>(__self);
    _obj->OnUpdate(deltaTime);   // 实际的 C++ 调用
}
```

4. **`CSharpGenerator`**：生成带有 `[LibraryImport]` 特性的 C# partial 类，指向相同的入口点：
```csharp
// Generated in JzRE.Runtime.Bindings.Gen.cs
private static partial class InternalCalls
{
    [LibraryImport("JzRE.Runtime", EntryPoint = "Script_OnUpdate")]
    internal static partial void OnUpdate(IntPtr __self, float deltaTime);
}
```

5. **手写的 C# partial 类**（`Source/Editor/Object.cs`）持有原生指针，并将其传递给内部调用：
```csharp
public abstract partial class Object : IDisposable
{
    internal IntPtr __unmanagedPtr;  // 指向原生 JzObject
    // ...
}
```

## 方向 2：C++ 调用 C#（托管虚表 / 函数指针）

这里更复杂 —— 它使得 C++ 可以调用 C# **对虚方法的重写**（例如用户脚本重写 `OnUpdate`）。

机制是 **带有注册的托管虚表（Managed VTable）**：

1. **C++ 代码生成** —— 对于每个拥有虚方法的类，`CppGenerator` 生成：
    - 为每个虚方法生成函数指针类型定义
    - 持有一个函数指针列表的 `_ManagedVTable` 结构体
    - 一个 `_SetManagedVTable` 导出函数，使得 C# 可以注册回调
    - 使得 C# 可以显式调用 C++ 基类实现的 `_CallBase` 桩函数
2. **C# 代码生成** —— 对于每个虚方法，`CSharpGenerator` 生成：
    - 一个 `[UnmanagedCallersOnly]` 静态回调方法，负责将 `IntPtr __self` 解析回托管包装对象，然后调用该虚方法
    - 一个 `unsafe` 静态构造函数，将函数指针注册到原生侧：
```csharp
static Script()
{
    unsafe
    {
        InternalCalls.SetManagedVTable(
            (IntPtr)(delegate* unmanaged[Cdecl]<IntPtr, void>)&ManagedCallback_OnUpdate,
            // ...
        );
    }
}
```
3. **跨平台 DLL 加载** —— 生成的代码包含一个 `NativeLibrary.SetDllImportResolver`，为每个平台选择正确的库扩展名（`.dll` / `.so` / `.dylib`）。

## 方向 3：原生持有 → 托管包装对象的创建（函数指针回调）

对于原生侧拥有生命周期、且当 C++ 对象被创建时 C# 包装对象也必须存在的对象，流程如下：

1. C# 注册函数指针回调到 C++ 侧，通过 `ScriptingEngine_RegisterInteropCallbacks`（在 `Source/Editor/NativeInterop.cs `中定义）。
2. 当原生侧创建了一个 `Script`，`ScriptingEngine` 调用 `CreateManagedObject(typeName, nativePtr, objectId)`，其具体实现为：
    - 通过 `Type.GetType()` + `Activator.CreateInstance()` 反射创建 C# 对象
    - 通过 `SetInternalValues()` 将 `__unmanagedPtr` 和 `__internalId` 写入托管对象中
    - 返回一个存储在 `JzObject::_gcHandle` 中的 `GCHandle` 指针

