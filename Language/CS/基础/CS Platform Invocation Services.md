P/Invoke（Platform Invocation Services）是 C# 中的一个特性，用于调用非托管代码中的函数，比如 C 或 C++ 编写的 DLL 中的函数。它允许 .NET 应用程序与操作系统的 API 或其他库进行交互，而这些库是以非托管代码形式存在的。

使用 P/Invoke 的主要步骤包括：
1. **声明外部函数**： 在 C# 中使用 `DllImport` 特性来声明外部函数。`DllImport` 特性指定了 DLL 的名称以及要调用的函数名称。
```c#
using System;
using System.Runtime.InteropServices;

class Program
{
    [DllImport("user32.dll", CharSet = CharSet.Auto)]
    public static extern int MessageBox(IntPtr hWnd, string text, string caption, uint type);

    static void Main()
    {
        MessageBox(IntPtr.Zero, "Hello, World!", "My Message Box", 0);
    }
}
```

2. **定义函数签名**： 确保你在 C# 中定义的函数签名与 DLL 中的函数签名完全匹配。这包括返回类型、参数类型、调用约定等。 
3. **管理数据类型转换**： 有些非托管数据类型与托管数据类型之间需要进行转换。例如，字符串、结构体和指针等数据类型的转换可能会涉及到额外的处理。



## Marshal

命名空间：`System.Runtime.InteropServices` ，提供了一个方法集合，这些方法用于分配非托管内存、复制非托管内存块、将托管类型转换为非托管类型，此外还提供了在与非托管代码交互时使用的其他杂项方法。

分配内存：
- `Marshal.AllocHGlobal(Int32)` ：	通过使用指定的字节数，从进程的非托管内存中分配内存。
- `Marshal.AllocHGlobal(IntPtr)` ：通过使用指向指定字节数的指针，从进程的非托管内存中分配内存。

拷贝：
- `Marshal.Copy(Int32[], Int32, IntPtr, Int32)` ：将数据从一维托管 32 位带符号整数数组复制到非托管内存指针。
- `Marshal.Copy(Single[], Int32, IntPtr, Int32)` ：将数据从一维托管单精度浮点数数组复制到非托管内存指针。

释放内存：
- `Marshal.FreeHGlobal(IntPtr)` ：释放以前从进程的非托管内存中分配的内存。


