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

1. 设置环境变量 `COMPlus_legacyCorruptedStateExceptionsPolicy=1`，让 CLR 回退到旧版异常处理行为。

