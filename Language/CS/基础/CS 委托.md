C# 的**委托**（Delegate）是一种类型安全的函数指针，它可以用来封装方法。委托类型定义了可以引用的特定方法签名，委托本身存储了对该方法的**引用**，可以在运行时调用这些方法。

多播委托允许将多个方法添加到同一个委托实例上（使用+=），并按顺序调用这些方法。


**事件**（Event）是基于委托（Delegate）的一种特殊机制，它为委托提供了封装和访问控制。可以将事件看作是委托的一个扩展，用于实现更为安全和结构化的事件驱动编程。

**基本概念**
- **声明**：事件使用委托类型声明，确保只有符合委托签名的方法可以订阅该事件。
- **订阅**：方法通过 `+=` 运算符订阅事件，即将方法添加到事件的委托列表中。
- **触发**：事件通常由类内部通过某些条件（例如用户输入、时间到达等）触发，并调用所有订阅的方法。
- **取消订阅**：可以通过 `-=` 运算符取消方法对事件的订阅。


```csharp
using System;

// 声明委托类型
public delegate void MyEventHandler(string message);

class Program
{
    // 声明事件
    public event MyEventHandler MyEvent;

    // 触发事件的方法
    public void TriggerEvent(string message)
    {
        // 判断事件是否有订阅者
        MyEvent?.Invoke(message);  // 触发事件
    }

    static void Main()
    {
        Program program = new Program();

        // 订阅事件
        program.MyEvent += PrintMessage;

        // 触发事件
        program.TriggerEvent("Hello, Event!");
    }

    // 事件处理方法
    static void PrintMessage(string message)
    {
        Console.WriteLine("Event triggered: " + message);
    }
}

```


**特点**
1. **封装**：事件为委托提供了封装，外部代码不能直接调用事件的委托实例，只能通过 `+=` 或 `-=` 订阅和取消订阅事件。
2. **安全**：事件的订阅和触发通常只允许内部类进行触发，外部类只能通过订阅和取消订阅来响应事件。这防止了外部直接调用事件的方法。
3. **多播**：类似于委托，事件也可以是多播委托，即可以有多个方法同时订阅一个事件，这些方法会按顺序被调用。


