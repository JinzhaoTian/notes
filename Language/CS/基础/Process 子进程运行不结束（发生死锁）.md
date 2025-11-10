
在项目中需要调用子程序，用process方式重定向标准输出到主进程，创建并启动子进程后发现exe运行结束后也process不结束，就像是被卡住了一样。


调查发现还是和 StandardOutput 有关，重定向的 StandardOutput 流可以用同步或异步读取，同步的方式有 `Read`、`ReadLine` 和 `ReadToEnd` 等方法，对进程的输出流执行同步读取操作。 这些同步读取操作直到关联的 Process 写入其 StandardOutput 流或关闭该流后才会完成。

`BeginOutputReadLine` 在 StandardOutput 流上启动异步读取操作，此方法为流输出启用指定的事件处理程序（请参阅 OutputDataReceived），并立即返回到调用方，调用方可以在流输出定向到事件处理程序时执行其他工作。处理异步输出的应用程序应调用 `WaitForExit` 方法以确保输出缓冲区已被刷新。

**同步读取操作在父进程和子进程之间会建立依赖关系，可能会导致死锁情况。** 这个依赖关系是：
- 当父进程从子进程的重定向流中读取数据时，它依赖于子进程。主进程等待读取操作，直到子进程写入流或关闭流。 
- 当子进程写入足够的数据来填充其重定向流时，它依赖于父进程。 子进程等待下一个写入操作，直到父进程从完整流中读取或关闭流。 

**当调用者和子进程相互等待对方完成操作并且都无法继续时，就会出现死锁情况**。 在使用时需要通过评估调用者和子进程之间的依赖关系来避免死锁。

## 正确使用

对于可执行程序 `Write500Lines.exe` ：
```c#
using System;
using System.IO;

public class SubExample
{
   public static void Main()
   {
      for (int ctr = 0; ctr < 500; ctr++)
         Console.WriteLine($"Line {ctr + 1} of 500 written: {ctr + 1/500.0:P2}");

      Console.Error.WriteLine("\nSuccessfully wrote 500 lines.\n");
   }
}
```


### 同步方式（会死锁）

在同步读取时，需要在 `p.WaitForExit` 之前调用 `p.StandardOutput.ReadToEnd` 来避免死锁情况。

```C#
using System;
using System.Diagnostics;

public class SyncMainExample
{
   public static void Main()
   {
      var p = new Process();  
      p.StartInfo.UseShellExecute = false;  
      p.StartInfo.RedirectStandardOutput = true;  
      p.StartInfo.FileName = "Write500Lines.exe";  
      p.Start();  

      // To avoid deadlocks, always read the output stream first and then wait.  
      string output = p.StandardOutput.ReadToEnd();  
      p.WaitForExit();

      Console.WriteLine($"The last 50 characters in the output stream are:\n'{output.Substring(output.Length - 50)}'");
   }
}
```

如果父进程在 `p.StandardOutput.ReadToEnd` 之前调用 `p.WaitForExit` 并且子进程写入足够的文本来填充重定向流，则可能会导致死锁情况。

此时，父进程将无限期地等待子进程退出，子进程将无限期地等待父进程从完整的 StandardOutput 流中读取数据。


### 异步方式（可避免死锁）

在异步读取时，可以通过创建两个线程并在单独的线程上读取每个流的输出来避免死锁情况。

```C#
using System;
using System.Diagnostics;
using System.Threading.Tasks;

public class AsyncMainExample
{
   public static void Main()
   {
      var p = new Process();  
      p.StartInfo.UseShellExecute = false;  
      p.StartInfo.RedirectStandardOutput = true;  
      p.StartInfo.RedirectStandardError = true;
      p.StartInfo.FileName = "Write500Lines.exe";  
      
      p.Start();  

      // To avoid deadlocks, use an asynchronous read operation on at least one of the streams.  
      Task<string> outputTask = process.StandardOutput.ReadToEndAsync(); 
      Task<string> errorTask = process.StandardError.ReadToEndAsync();

      // 等待进程完成
      p.WaitForExit();

      // 等待输出任务完成 
      string output = outputTask.Result;
      string error = errorTask.Result;

      Console.WriteLine($"The last 50 characters in the output stream are:\n'{output.Substring(output.Length - 50)}'");
      Console.WriteLine($"\nError stream: {error}");
  
   }
}
```