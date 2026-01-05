
## `IDisposable`

`IDisposable` 是 C# 用于显式释放[非托管内存](CS%20托管内存与非托管内存.md)的一个接口，它定义了一种标准机制，让对象的使用者能够主动释放对象占用的资源，而不是等待垃圾回收器自动回收。

```csharp
public interface IDisposable
{
    void Dispose();
}
```

### 核心目的

1. **释放非托管资源**（如文件句柄、数据库连接、网络连接、GDI+对象等）
2. **及时释放内存**，特别是大对象
3. **确定性清理**（在确定的时间点执行清理）


### 使用示例

1. **简单实现**：
```csharp
public class ResourceHolder : IDisposable
{
    private FileStream _fileStream;
    
    public ResourceHolder(string filePath)
    {
        _fileStream = new FileStream(filePath, FileMode.Open);
    }
    
    public void Dispose()
    {
        if (_fileStream != null)
        {
            _fileStream.Dispose();
            _fileStream = null;
        }
    }
}
```

2. **显式调用 `.Dispose()`**：
```csharp
var resource = new ResourceHolder("file.txt");
try
{
    // 使用资源
}
finally
{
    resource.Dispose();
}
```

3. **使用 `using` 语句**（推荐）：
```csharp
using (var resource = new ResourceHolder("file.txt"))
{
    // 使用资源
    // 离开using块时自动调用Dispose()
}
```

4. **使用 `using` 声明**（C# 8.0+）：
```csharp
using var resource = new ResourceHolder("file.txt");
// 使用资源
// 离开作用域时自动调用Dispose()
```

### 重要原则

1. **实现 `IDisposable` 的类通常也应该实现 `Finalizer`**（除非基类已经实现）
2. **`Dispose` 方法应该可以安全地多次调用**（幂等）
3. **释放资源后应将对象标记为不可用**
4. **如果类包含实现了 `IDisposable` 的字段，类本身也应该实现 `IDisposable`**
5. **避免在 `Dispose` 中抛出异常**
6. **继承时，记得调用基类的 `Dispose` 方法**





