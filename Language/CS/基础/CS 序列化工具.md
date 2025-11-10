
### Json

1. [Newtonsoft.Json](https://www.nuget.org/packages/Newtonsoft.Json)
   Newtonsoft.Json 是 .NET 生态系统中最流行和广泛使用的 JSON 序列化和反序列化工具，具有丰富的功能和灵活的 API ，支持自定义序列化和反序列化过程。性能表现良好，具有广泛的社区支持，老牌序列化工具，支持 .NET Framework 3.5 以上版本。![](imgs/Pasted%20image%2020240305155951.png)
2. System.Text.Json
   System.Text.Json 是 .NET Core3.1 之后官方提供的 JSON 序列化和反序列化库，与 .NET Core 集成紧密，具有较低的内存占用和较好的性能，支持异步操作，并且是 .NET Core 开发的首选选择之一。
3. [Swifter.Json](https://www.nuget.org/packages/Swifter.Json)
   Swifter.Json 是 .Net 平台上一个功能强大，简单易用，稳定及高性能的 Json 序列化和反序列化工具。性能方面相比 Newtonsoft.Json 提高了 5 到 10 倍左右。![](imgs/Pasted%20image%2020240305160302.png)
4. System.Runtime.Serialization.Json
   System.Runtime.Serialization.Json 是 .NET Framework 内置的库，用于序列化和反序列化 JSON数据。它支持 DataContract 属性和 WCF 协定，但在性能方面可能不如其他库。比较老的项目可以使用它。


### Binary

1. [MessagePack](https://www.nuget.org/packages/MessagePack)
   MessagePack是一种高效的二进制序列化格式，可以将对象序列化为紧凑的字节流，也可以将字节流反序列化为对象。它具有比JSON更小的序列化尺寸和更快的序列化速度。MessagePack支持多种编程语言，并且可以在跨语言的应用程序中使用。![](imgs/Pasted%20image%2020240305160726.png)