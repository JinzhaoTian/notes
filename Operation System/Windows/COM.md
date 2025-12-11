COM（Component Object Model，组件对象模型）是微软在 1990 年代提出的一套二进制级别的软件组件接口标准，用于实现跨语言、跨进程、跨机器的软件组件复用和交互。

## 主要特性

1. **语言无关性**：COM 组件可以用 C++、VB、Delphi、C# 等多种语言编写，不同语言编写的组件可以相互调用。
2. **位置透明性**：组件可运行在：
    - 同一进程内（DLL）
    - 不同进程间（EXE + 进程间通信）
    - 不同机器间（DCOM，分布式 COM）
3. **二进制标准**：定义了一套内存布局和函数调用约定，确保不同编译器生成的二进制代码能交互。
4. **版本兼容性**：通过接口继承和查询机制，支持组件升级而不破坏现有客户端。


## 关键技术概念

1. **接口（Interface）**：
	- COM 的核心抽象，是一组相关函数的集合
	- 所有 COM 接口都继承自 `IUnknown`
	- 使用虚拟函数表（vtable）实现
	- 每个接口有唯一的 GUID 标识
2. **`IUnknown` 接口**：所有 COM 接口的基类，包含三个核心方法：
```
interface IUnknown {
    virtual HRESULT QueryInterface(REFIID riid, void** ppv) = 0;
    virtual ULONG AddRef() = 0;
    virtual ULONG Release() = 0;
};
```
- **核心方法**：
	- **`QueryInterface`**：查询组件是否支持特定接口
	- **`AddRef`**
	- **`Release`**：引用计数，实现生命周期管理

3. **类工厂（Class Factory）**：创建 COM 对象实例的专门对象，实现 `IClassFactory` 接口。
4. **注册表（Registry）**：Windows 注册表存储 COM 组件的 CLSID、接口IID、路径等信息，实现组件的自动发现。

## 相关技术与演变

1. **自动化（Automation）**：通过 `IDispatch` 接口支持脚本语言（如 VBScript）
2. **ActiveX控件**：基于 COM 的可视化控件
3. **OLE（对象链接与嵌入）**：基于 COM 的文档复合技术
4. **COM+**：增加了事务、安全、队列等企业级服务
5. **.NET取代**：虽然 .NET 提供了更好的组件模型，但 COM 仍在 Windows 底层广泛使用