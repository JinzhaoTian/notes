特性（Attribute）`HandleProcessCorruptedStateExceptions` 允许方法捕获损坏状态异常（Corrupted State Exceptions，CSE），这些通常是由 CLR 抛出的严重异常，表示进程状态可能已损坏。

典型的损坏状态异常包括：
1. **AccessViolationException**（访问冲突）
2. StackOverflowException（堆栈溢出）
3. SEHException（结构化异常处理异常）


> [!info] 产生背景
> 从 .NET Framework 4.0 开始，CLR 默认不再允许托管代码捕获这些严重异常，因为这些异常通常表示进程处于不可恢复的状态，继续执行可能导致进一步损坏，使用此特性可以覆盖默认行为，允许捕获这些异常。


## 使用注意事项

### 优点

1. 提供故障恢复机制
2. 防止应用程序因单一几何计算失败而完全崩溃

### 风险

1. 掩藏了真正的严重问题
2. 可能导致内存泄漏或进一步的状态损坏
3. 应在严格控制的范围内使用


## 替代方案

```cs
// 更现代的.NET Core/5+中，可以使用：
[SupportedOSPlatform("windows")]
[UnmanagedCallersOnly]
// 或者使用try-catch的特定配置
```