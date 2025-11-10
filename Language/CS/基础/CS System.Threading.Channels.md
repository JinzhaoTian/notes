System.Threading.Channels 是 .NET Core 3.0 引入的一种新的集合类型，具有异步API、高性能和线程安全等特点。它可以用来创建消息队列，进行数据的生产和消费。

Channel 提供了 Writer 和 Reader API，分别对应消息的生产者和消费者，使得 Channel 更加简洁和易用。

Channel 实际上是使用 `ConcurrentQueue` 做的封装，使用起来更方便，对异步更友好。


## 使用示例

1. **创建 `Channel`**：
```cs
// 创建有限容量的 Channel
var channel = Channel.CreateBounded<string>(100);

// 创建无限容量的 Channel
var channel = Channel.CreateUnbounded<string>();
```

2. **生产数据**：
```cs
// 1. WriteAsync
await channel.Writer.WriteAsync("hello");

// 2. TryWrite
var status = await channel.Writer.TryWrite("hello"); // 如果写入数据失败时会返回 false

// 3. WaitToWriteAsync
await channel.Writer.TryWrite("hello"); // 非阻塞，直到 Channel 允许写入新的数据时返回 true
```

3. **消费数据**：
```cs
// 1. ReadAsync
var item = await channel.Reader.ReadAsync();

// 2. TryRead

// 3. WaitToReadAsync
```

