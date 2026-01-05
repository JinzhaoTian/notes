.NET 的垃圾回收（Garbage Collection，GC）是一种自动内存管理机制，负责在托管堆（Managed Heap）上分配和回收对象，避免内存泄漏和悬空指针。

## 核心原理

1. **代际假说**（Generational Hypothesis）
	- **弱代假说**：大多数对象生命周期很短
	- **强代假说**：存活下来的对象往往能存活更久
	- 基于此，.NET 将堆分为三代：
	    - **第0代**：新创建的对象（生命周期最短）
	    - **第1代**：从第0代存活下来的对象
	    - **第2代**：从第1代存活下来的对象（生命周期最长）

2. **托管堆结构**：
```
Small Object Heap (SOH)         Large Object Heap (LOH)
[ Gen0 | Gen1 | Gen2 ]           [ 单独管理 ]
(<85KB的对象)                     (≥85KB的对象)
```


## 工作流程

1. **标记阶段**：
```csharp
// GC会标记所有"可达"对象
public class Example
{
    private object _reference;
    
    void Method()
    {
        var obj1 = new object();  // 创建对象
        var obj2 = new object();
        _reference = obj2;         // obj2被根引用
        // obj1在方法结束时变为不可达
    }
}
```

2. **整理阶段**：
	- 移动存活对象，消除内存碎片
	- 更新对象引用指针
3. **清扫阶段**：
	- 回收未标记对象的内存


## 类型

1. **Workstation GC**：
	- **并发模式**：GC 在后台线程运行，最小化暂停时间
	- **非并发模式**：完全停止应用程序线程

2. **Server GC**：
	- 每个 CPU 核心有独立的堆和 GC 线程
	- 为高吞吐量服务器应用优化


## 大对象堆（LOH）特殊处理

```csharp
public class LOHExample
{
    void LargeAllocation()
    {
        // 分配85KB以上的对象进入LOH
        var largeArray = new byte[1024 * 100]; // 100KB
        
        // LOH特点：
        // 1. 只在进行完整GC（Gen2收集）时回收
        // 2. 默认不进行内存整理（.NET 4.5.1+可配置）
        // 3. 可能产生内存碎片
    }
}
```


## 优化技巧

1. 避免不必要的对象分配
```csharp
// 不好的做法：产生字符串分配
string result = "";
for (int i = 0; i < 100; i++)
{
    result += i.ToString(); // 每次拼接创建新字符串
}

// 好的做法：使用StringBuilder
var sb = new StringBuilder();
for (int i = 0; i < 100; i++)
{
    sb.Append(i);
}
string result = sb.ToString();
```

2. 及时释放非托管资源
```csharp
public class ResourceHolder : IDisposable
{
    private IntPtr _nativeResource;
    
    // 实现Dispose模式
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this); // 阻止终结器调用
    }
    
    protected virtual void Dispose(bool disposing)
    {
        if (disposing)
        {
            // 释放托管资源
        }
        // 释放非托管资源
        if (_nativeResource != IntPtr.Zero)
        {
            // 调用Native方法释放
            _nativeResource = IntPtr.Zero;
        }
    }
    
    ~ResourceHolder() // 终结器
    {
        Dispose(false);
    }
}
```

3. 使用弱引用
```csharp
public class WeakReferenceExample
{
    private WeakReference<LargeObject> _weakRef;
    
    void Process()
    {
        var largeObj = new LargeObject();
        _weakRef = new WeakReference<LargeObject>(largeObj);
        
        // ... 使用largeObj
        
        // 当内存紧张时，GC可以回收LargeObject
        // 但仍可以通过弱引用尝试访问
        if (_weakRef.TryGetTarget(out var obj))
        {
            // 对象还在，可以使用
        }
    }
}
```


## 最新改进（.NET Core/.NET 5+）

1. 区域性GC
	- 允许指定对象关联性，优化 NUMA 架构

2. 可扩展GC
	- 更好的多核扩展性

3. 容器感知
	- 自动检测容器内存限制
