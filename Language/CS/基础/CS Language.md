[编程之路-学习C#](https://www.cjavapy.com/category/66/)

C#是由C和C++衍生出来的一种安全的、稳定的、简单的、优雅的**面向对象编程语言**。它在继承C和C++强大功能的同时去掉了一些它们的复杂特性（例如没有宏以及不允许多重继承）。C#综合了VB简单的可视化操作和C++的高运行效率，以其强大的操作能力、优雅的语法风格、创新的语言特性和便捷的面向组件编程的支持成为.NET开发的首选语言。

第一段C#代码：
```C#
using System;                                 // 使用命名空间

namespace Simple                              // 声明新的命名空间
{
	class Program                             // 声明一个新类
	{
		static void Main()                    // 声明一个名称为Main的成员方法
		{
			Console.WriteLine("Hi there");    // Main方法体
		}
	}
}
```


# C# 基础语法

## 1. 变量和类型

### 1.1 预定义数据类型

C#有16种预定义数据类型，其中**简单类型**13种，包括bool、char（Unicode字符）、sbyte、byte、short、ushort、int、uint、long、ulong等，以及decimal（高精度小数类型）、float、double。
![](_imgs/Pasted%20image%2020230809222248.png)

**非简单类型**有：
- object：所有其他类型的基类。
- string：Unicode字符数组。
- dynamic：使用动态语言编写的程序集时使用。

所有预定义类型都直接映射到底层的 .NET 类 型，C#的类型名称就是 .NET 类型的别名，所以使用 .NET 的类型名称也符合 C# 的语法（但是并不鼓励这么做）。


### 1.2 用户定义类型

常见的有：类类型（class）、结构类型（struct）、数组类型（array）、枚举类型（enum）、**委托类型**（delegate）、**接口类型**（interface）。



### 1.3 值类型和引用类型

![](_imgs/Pasted%20image%2020230809223835.png)
要注意的是char、float、double竟然是引用类型。


### 1.4 dynamic

dymamic 关键字代表一个特定的C#类型，它知道如何在运行时解析自身。编译时，编译器不会对dynamic 类型的变量做类型检查。相反，它将与该变量及该变量的 操作有关的所有信息打包。在运行时，会对这些信息进行检查，以确保它与变量所代表的实际类 型一致。否则，将在运行时抛出异常。

## 2. 表达式和语句

### 2.1 using语句

1. 引入命名空间
```C#
using System;
```

2. using static：指定无需指定类型名称即可访问其静态成员的类型
```C#
using static System.Math;var = PI;        // 直接使用System.Math.PI
```

3. 别名
```C#
using AcadApp = Autodesk.AutoCAD.ApplicationServices.Application;
```

4. using 语句：将实例与代码绑定
```C#
using (AcadDatabase adb = AcadDatabase.Active())
{
	// ...
}
```
在代码中，如果调用了**非托管资源**，就必须用完之后要去显式释放它，如果不去释放它，那么可能就会造成内存泄漏。使用using关键字就可以比较方便的让系统自动释放资源。

如果想要使用这个语法糖，必须要一个标准的释放非托管资源的类实现IDisposable接口，如：
```C#
public class AcadDatabase : IDisposable
{
	// ...
	public void Dispose()
	{
		// ...
	}
}
```

其中，IDisposable 接口是一个空接口，源码如下：
```C#
namespace System
{
    public interface IDisposable
    {
        void Dispose();
    }
}
```


## 3. 文档注释

除了单行注释（ `//` ），多行注释（ `/* */` ），还有文档注释，与Java类似。文档注释的主要格式为：
```C#
/// <sumary>                   <- 类开始的XML标签
/// This class does. 
/// </summary>                 <- 关闭XML标签
class Program
{
	 /// <summary>             <- 字段开始XML标签
	 /// Fields is used to hold the value of ...
	 /// </summary>            <- 关闭XML标签
    public int Fieldi = 10;
	...
}
```
文档注释允许我们以 XML 元素的形式在程序中包含文档，Visual Studio 会帮助我们插入元素，以及从源文件中读取它们并复制到独立的 XML 文件中，包括如下步骤：
- 使用 Visual Studio 来产生嵌人了 XML 的源文件。
- Visual Studio从源文件中读取 XML 并且复制 XML 代码到新的文件。
- **文档编译器**可以获取 XML 文件并且从它产生各种类型的文档文件。





# [C# 面向对象](基础/CS%20面向对象.md)

面向对象的行为，包括继承原则（单继承多继承）等等。（顺便总结一下C++，Java，Python的）



# C# 泛型

# C# 常用框架

## System.String

String 对象不可变，每次使用 System.String 类中的方法之一，都要在内存中新建字符串对象，这就需要为新对象分配新空间。 在需要重复修改字符串的情况下，与新建 String 对象关联的开销可能会非常大。 

## System.Text

若要修改字符串（而不新建对象），可以使用 System.Text.StringBuilder 类。 例如，如果在循环中将许多字符串连接在一起，使用 StringBuilder 类可以提升性能。




## System.Object

## System.IO

### 管道数据

管道为进程间通信提供了一种可能，管道分为两种，一种是匿名管道，另一种是命名管道。
- **匿名管道** - 匿名管道只提供在本地电脑进程间的通信。匿名管道比命名管道花费的开销更少，但提供的服务也比命名管道的少。匿名管道是单向的，而且不能用于网络通信。匿名管道只支持单服务器实例。
	- 匿名管道不支持消息传输模式（PipeTransmissionMode.Message），仅支持字节传输模式（PipeTransmissionMode.Byte）。
- **命名管道** - 命名管道既可以用于本地进程间的通信，也可以用于网络通信。
	- 可以支持直接传输模式（PipeTransmissionMode.Byte）
	- 可以支持消息传输模式（PipeTransmissionMode.Message）

```C#
// Pipeline Server
pubilc void Server() 
{ 
	NamedPipeServerStream pipeServer = 
		new NamedPipeServerStream("testpipe", PipeDirection.InOut, numThreads); 
	int threadId = Thread.CurrentThread.ManagedThreadId; 
	
	// Wait for a client to connect 
	pipeServer.WaitForConnection(); 
	try 
	{ 
		// Read the request from the client. Once the client has 
		// written to the pipe its security token will be available. 
		
		StreamString ss = new StreamString(pipeServer); 
		
		// Verify our identity to the connected client using a 
		// string that the client anticipates. 
		
		ss.WriteString("I am the one true server!"); 
		string filename = ss.ReadString(); 
		
		// Read in the contents of the file while impersonating the client.
		ReadFileToStream fileReader = new ReadFileToStream(ss, filename); 
		
		// Display the name of the user we are impersonating. 
		Console.WriteLine("Reading file: {0} on thread[{1}] as user: {2}.", 
			filename, threadId, pipeServer.GetImpersonationUserName());
		pipeServer.RunAsClient(fileReader.Start); 
	} 
	// Catch the IOException that is raised if the pipe is broken 
	// or disconnected. 
	catch (IOException e) 
	{ 
		Console.WriteLine("ERROR: {0}", e.Message); 
	} 
	pipeServer.Close(); 
}
```


```C#
// Pipeline Client
public void Client() 
{ 
	var pipeClient = 
		new NamedPipeClientStream(".", "testpipe", 
			PipeDirection.InOut, PipeOptions.None, 
			TokenImpersonationLevel.Impersonation); 
	
	pipeClient.Connect(); 
	var ss = new StreamString(pipeClient); 
	
	// Validate the server's signature string. 
	if (ss.ReadString() == "I am the one true server!") 
	{ 
		// The client security token is sent with the first write. 
		// Send the name of the file whose contents are returned 
		// by the server. 
		ss.WriteString("c:\\textfile.txt"); 
		
		// Print the file to the screen. 
		Console.Write(ss.ReadString()); 
	} 
	else 
	{ 
		Console.WriteLine("Server could not be verified."); 
	} 
	pipeClient.Close(); 
}
```


## System.ComponentModel

### BackgroundWorker

耗时操作在长时间运行时会导致用户界面 (UI) 处于停止响应状态，用户在这操作期间无法进行其他的操作，造成非常差的用户体验。

为了不使UI层处于停止响应状态，则可以使用 BackgroundWorker 类方便地解决这类问题。这个后台的线程处理，可以很好的实现常规操作的同时，还可以及时通知UI当前处理信息的进度等。

如，做一个界面的进度条，在后台线程中进行耗时运算，同时刷新界面上的进度条。

```C#
 public class BackgroundWorkerExampleClass
 {
	BackgroundWorker backgroundWorker = null;
	
	private void InitializeBackgroundWorker()
	{
		backgroundWorker = new BackgroundWorker();
		backgroundWorker.DoWork += 
			new DoWorkEventHandler(backgroundWorker1_DoWork);
		backgroundWorker.RunWorkerCompleted += 
			new RunWorkerCompletedEventHandler( backgroundWorker_RunWorkerCompleted); 
		backgroundWorker.ProgressChanged += 
			new ProgressChangedEventHandler( backgroundWorker_ProgressChanged);
	}
	
	// This event handler is where the actual, 
	// potentially time-consuming work is done. 
	private void backgroundWorker_DoWork(object sender, DoWorkEventArgs e) 
	{ 
		// Get the BackgroundWorker that raised this event. 
		BackgroundWorker worker = sender as BackgroundWorker; 
		
		// Assign the result of the computation 
		// to the Result property of the DoWorkEventArgs 
		// object. This is will be available to the 
		// RunWorkerCompleted eventhandler. 
		e.Result = ComputeFibonacci((int)e.Argument, worker, e); 
	}
	
	// This event handler deals with the results of the 
	// background operation. 
	private void backgroundWorker1_RunWorkerCompleted( object sender, RunWorkerCompletedEventArgs e) 
	{ 
		// First, handle the case where an exception was thrown. 
		if (e.Error != null) 
		{ 
			MessageBox.Show(e.Error.Message); 
		} 
		else if (e.Cancelled) 
		{ 
			// Next, handle the case where the user canceled 
			// the operation. 
			// Note that due to a race condition in 
			// the DoWork event handler, the Cancelled 
			// flag may not have been set, even though 
			// CancelAsync was called. 
			resultLabel.Text = "Canceled"; 
		} 
		else 
		{ 
			// Finally, handle the case where the operation 
			// succeeded. 
			resultLabel.Text = e.Result.ToString(); 
		} 
		
		// Enable the UpDown control. 
		this.numericUpDown11.Enabled = true; 
		// Enable the Start button. 
		startAsyncButton.Enabled = true; 
		// Disable the Cancel button. 
		cancelAsyncButton.Enabled = false; 
	} 
	
	// This event handler updates the progress bar. 
	private void backgroundWorker1_ProgressChanged(object sender, ProgressChangedEventArgs e) 
	{ 
		this.progressBar.Value = e.ProgressPercentage; 
	}

}
```


# LINQ

语言集成查询（Language Integrated Query，Linq），是一种使用类似 SQL 语句操作多种数据源的功能。可以使用 C# 查询 access 数据库、.net 数据集、xml 文档以及实现了 `IEnumerable` 或`IEnumerable<T>` 接口的集合类（如 List，Array，SortedSet，Stack，Queue 等，可以进行遍历的数据结构都会集成该类）。从.net framework 3.5中开始引入，能够提升程序数据处理能力和开发效率，具有集成性、统一性、可扩展性、抽象性、说明式编程、可组成型、可转换性等优势。

## 使用场景

- LINQ to Objects - 直接将 LINQ 查询与任何 `IEnumerable` 或 `IEnumerable<T>` 集合一起使用，而不使用中间 LINQ 提供程序或 API。可以使用 LINQ 来查询任何可枚举的集合，例如 `List<T>`、`Array` 或 `Dictionary<TKey,TValue>`。 
- LINQ to Entities -  将 LINQ 查询转换为**命令目录树**查询，针对实体框架执行这些查询，并返回可同时由实体框架和 LINQ 使用的对象。
	- 从 `ObjectQuery<T>` 构造 ObjectContext 实例。
	- 通过使用 `ObjectQuery<T>` 实例在 C# 中编写 LINQ to Entities 查询。
	- 将 LINQ 标准查询运算符和表达式将转换为命令目录树。
	- 对数据源执行命令目录树表示形式的查询，执行过程中在数据源上引发的任何异常都将直接向上传递到客户端。
	- 将查询结果返回到客户端。
- LINQ to XML - 提供文档对象模型（DOM）的内存中文档修改功能，既能修改内存中的xml，也可以修改从文件中加载的。
- LINQ to DataSet - 提供程序查询ADO.NET数据集中的数据。
- LINQ to SQL - 提供程序查询和修改Sql Server数据库中的数据，将应用程序中的对象模型映射到数据库表。

## 查询语法

- select - 基于查询结果返回需要的值或字段，并能够对返回值指定类型。
- from - 用来标识查询的数据源。
- where - 用来指定筛选的条件。
- order by - 用来排序。
- group by - 用来对查询结果进行分组。
- join - 用于联合查询


## 方法语法

常见的有Sum、Max、Min、Average等。

```C#
var queryInfo = from score in scores select score;

var averqge = queryInfo.Average();               // 内部方法
var maxScore = queryInfo.Max();
var minScore = queryInfo.Min();

```


# C# 并发编程

[C#（99）：三、并行编程 - Task同步机制。TreadLocal类、Lock、Interlocked、Synchronization、ConcurrentQueue以及Barrier等 ](https://www.cnblogs.com/springsnow/p/9409477.html)

# C# 异步编程

使用异步编程，方法调用是在后台运行，并不会阻塞调用线程。.NET Framework有不同的异步模式：
- 异步模式
- 基于事件的异步模式：Windows Forms和WPF
- 基于任务的异步模式（TAP）

HTTP请求同步调用：
```C#
private static void SynchronizedAPI()
{
	using (var client = new WebClient())
	{
		string content = client.DownloadString(url);
		Console.WriteLine(content);
	}
}
```


HTTP请求异步模式：
```C#
private static void AsynchronizedPattern()
{
	WebRequest request = WebRequest.Create(url);
	IAsyncResult result = request.BeginGetResponse(ReadResponse, null);
	
	void ReadRequest(IAsyncResult ar)
	{
		using (WebResponse response = request.EndGetResponse(ar))
		{
			Stream stream = request.GetResponseStream();
			var reader = new StreamReader(stream);
			string content = reader.ReadToEnd();
			Console.WriteLine(content);
		}
	}
}
```


基于事件的异步：
```C#
private static void EventBasedAsyncPattern()
{
	using (var client = new WebClient())
	{
		client.DownloadStringCompleted += (sender, e) =>
		{
			Console.WriteLine(e.Result.Substring(0, 100));
		};
		client.DownloadStringAsync(new Uri(url));
	}
}
```


基于任务的异步：
```C#
private static async Task TaskBasedAsyncPatternAsync()
{
	using (var client = new WebClient())
	{
		string content = await client.DownloadStringTaskAsync(url);
		Console.WriteLine(content);
	}
}


static sync Task Main()
{
	await TaskBasedAsyncPatternAsync();
}
```


## 延续任务

```C#
// UI
private async void BtnStartReview_Click(object sender, RoutedEventArgs e)
{
    await reviewViewModel.StartIFCExaminationAsync();

    reviewViewModel.UpdateCurrentViewModelByResult();
}

public async Task StartIFCExaminationAsync()
{
    await IfcReviewService.SendReviewAsync()
                .ContinueWith(res => { IfcReviewInfo = res.Result; });
}
```



# .Net Framework

.NET框架由三个部分组成：
- 执行环境：公共语言运行库（Common Language Runtime，CLR）
	- 内存管理和垃圾收集（GC，Garbage Collector）
	- 代码安全验证
	- 代码执行、线程管理及异常处理
- 基类库（Base Class Library，BCL），.NET框架使用的一个大的类库，有时也称为框架类库（Framework Class Library，FCL）。
	- 通用基础类
	- 集合类
	- 线程和同步类
	- XML类
- 编程工具
	- Visual Studio
	- .NET兼容的编译器
	- 调试器
	- Web开发服务器端技术，如ASP.NET或WCF。

.NET语言的编译器接受源代码文件，并生成名为**程序集**（Assembly）的输出文件。程序集要么是可执行的（`*.exe`），要么是DLL（`*.dll`）。尽管 .NET 的程序集文件与非托管的 Windows 二进制文件采用相同的文件扩展名（`*.dll`），但它们的内部完全不同。.NET程序集的代码是一种中间语言CIL（Common Intermediate Language，通用中间语言），CIL直到被调用运行时才会被编译成本机代码，CIL的代码只在需要的时候由JIT编译，然后它就被缓存起来。

 > 相比之下Java也是首先将代码编译成一个中间代码，称之为字节码（`.class`），运行时Java有两种运行模式：解释器（interpreter）模式和即时编译（Just-in-time compilation，JIT）模式。Java会根据代码运行情况转换到JIT的执行，解释器和JIT编译器混合配合工作。

CIL被编译成本机代码后，CLR负责对其进行管理，有两个重要概念：
- **托管代码（Managed Code）**：为 .NET Framework或 .NET Core 等编写的代码。托管代码由CLR负责管理内存分配、垃圾回收、异常处理等任务，以确保代码的安全性和可靠性。托管代码受到CLR的严格控制，开发者不需要手动管理内存或资源。
- **非托管代码（Unmanaged Code）**：非托管代码是指在非托管执行环境中运行的代码，如使用C、C++等编程语言编写的代码。在非托管环境中，开发者需要手动管理内存分配和资源释放，因为没有CLR提供的自动垃圾回收和资源管理功能。

 > 在Java中，所有的代码都被编译成字节码（bytecode），它在 JVM 上执行。开发者不需要直接管理内存和资源，JVM 负责垃圾回收和资源管理。因此 Java 中的代码都是类似于托管代码的概念，Java 也可以与一些涉及本机代码的技术进行互操作，例如使用 Java Native Interface（JNI）来调用本机库，但这种情况在 Java 中相对较少见，而且不是主要的编程模型。
 
.NET平台支持C#、C++、Visual Basic、Jscript、COBOL 等编程语言，.NET Core可以运行在Windows、Linux、macOS上。


运行命令 `dotnet --info` 查看 SDK 版本和运行时版本：![](_imgs/Pasted%20image%2020230921112625.png)




# 非托管代码调用

## C#调用C++代码步骤

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
	- 在Visual Studio中，选择 `项目 -> 属性` ，然后在配置类型中设置为动态链接库(.dll)![](_imgs/Pasted%20image%2020230921143540.png)
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



# 参考