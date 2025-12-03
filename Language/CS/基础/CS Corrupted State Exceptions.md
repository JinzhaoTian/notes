损坏状态异常（Corrupted State Exceptions，CSE），这些通常是由 CLR 抛出的严重异常，表示进程状态可能已损坏。

典型的损坏状态异常包括：
1. **AccessViolationException**（访问冲突）
2. StackOverflowException（堆栈溢出）
3. SEHException（结构化异常处理异常）


> [!info] 产生背景
> 从 .NET Framework 4.0 开始，CLR 默认不再允许托管代码捕获这些严重异常，因为这些异常通常表示进程处于不可恢复的状态，继续执行可能导致进一步损坏，使用特性（Attribute）`HandleProcessCorruptedStateExceptions` 可以覆盖默认行为，允许捕获这些异常。

> [!warning] 从 .NET Core 1.0 开始 CSE 完全无法由托管代码捕获
> 从 .NET Core 1.0 开始引入一项安全策略变更，在 .NET Core 1.0+ 以及后续 .NET 5/6/7/8 版本中，损坏状态异常（Corrupted State Exceptions，CSE）完全无法由托管代码捕获，公共语言运行时（CLR）不会将此类异常传递给托管代码。
> 
> 因此，进程遇到无法处理的 CSE 时会直接终止。
> 
> 其**设计目的**是：.NET Framework 4.0+ 虽然提供了向后兼容，但微软很快意识到这可能导致安全漏洞，CSE 意味着进程内存可能已损坏，捕获它可能让攻击者利用内存破坏漏洞，最安全的做法是让进程立即终止。


## 处理策略

如果进程因无法捕获的 `AccessViolationException` 而崩溃，这通常是 .NET 运行时在强制执行一项重要的安全策略，以防止在内存可能已损坏的情况下继续运行，从而避免更严重的安全漏洞。


此时的处理策略有两种：
### （临时方案）启用旧版兼容策略

设置环境变量 `COMPlus_legacyCorruptedStateExceptionsPolicy=1`，让 CLR 回退到旧版异常处理行为：

```bash
set COMPlus_legacyCorruptedStateExceptionsPolicy=1
dotnet YourApp.dll
```

```xml
<PropertyGroup>
  <StartupEnvironmentVariables>
    <StartupEnvironmentVariable Name="COMPlus_legacyCorruptedStateExceptionsPolicy" Value="1" />
  </StartupEnvironmentVariables>
</PropertyGroup>
```

**优点**：
1. 配置简单，可暂时让 `catch` 块捕获 `AccessViolationException`

**缺点**：
1. 严重安全风险：内存可能已损坏，程序后续行为不可预测，可能掩盖严重漏洞。
2. 不推荐用于生产环境。



### （推荐方案）进程隔离架构

将可能抛出 `AccessViolationException` 的高风险操作（调用 Xbim 等非托管库）放入独立的子进程中执行，主进程通过进程间通信 (IPC) 获取结果。

1. **创建一个独立的工作进程控制台应用**，专门负责调用高风险的非托管库。
2. 在主进程（如您的 ASP.NET Core 应用）中使用进程间通信（IPC），如命名管道、gRPC、TCP或简单的标准输入/输出，向工作进程发送请求并接收结果。
3. **在工作进程中**，可以尝试使用 `try-catch`。如果 CSE 发生，工作进程会崩溃退出，但主进程可以通过 IPC 通道的异常或退出码检测到失败，并采取相应措施（如记录日志、返回错误信息给用户、重试或启动新的工作进程），而自身保持稳定。


