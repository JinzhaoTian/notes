
## `DispatcherTimer`

`DispatcherTimer` 是 WPF、UWP、WinUI 等基于 **Dispatcher** 模型的 UI 框架中常用的一个计时器类，其核心特点是：**Tick 事件会在 UI 线程上触发**。

### 与其他计时器的核心区别

|计时器|线程模型|能否直接更新 UI|
|---|---|---|
|`DispatcherTimer`|Tick 在 UI 线程触发|✅ 可以直接更新 UI 控件|
|`System.Timers.Timer`|Tick 在线程池线程触发|❌ 需要通过 `Dispatcher.Invoke` 封送到 UI 线程|
|`System.Threading.Timer`|Tick 在线程池线程触发|❌ 同上|
### 用法示例

```csharp
using System.Windows.Threading;

// 创建计时器
DispatcherTimer timer = new DispatcherTimer();
timer.Interval = TimeSpan.FromSeconds(1);  // 1秒触发一次
timer.Tick += Timer_Tick;                   // 注册事件
timer.Start();                              // 开始

private void Timer_Tick(object sender, EventArgs e)
{
    // 这里的代码运行在 UI 线程上
    myLabel.Content = DateTime.Now.ToString("HH:mm:ss");
}

// 停止计时器
timer.Stop();
```

### 重要特性

#### 1. 不保证精确计时

`DispatcherTimer` 的 Tick 事件依赖 UI 线程的消息循环。如果 UI 线程繁忙（执行耗时操作），Tick 事件会被延迟执行，甚至跳过中间的 Tick。

```csharp
// 假设间隔 100ms，但 UI 线程阻塞了 500ms
// Tick 最多只会触发一次（不会积压）
```

#### 2. 优先级可控制

可以通过 `DispatcherTimer` 的构造函数指定 `DispatcherPriority`：

```csharp
DispatcherTimer timer = new DispatcherTimer(
    DispatcherPriority.Background  // 优先级较低，其他 UI 事件优先执行
);
timer.Interval = TimeSpan.FromMilliseconds(100);
timer.Tick += OnTick;
timer.Start();
```

常用优先级：`Normal`（默认）、`Background`、`Input` 等。

#### 3. 随 Dispatcher 生命周期自动回收

当创建 `DispatcherTimer` 的窗口或控件被销毁、Dispatcher 停止时，计时器也会随之停止并释放资源。