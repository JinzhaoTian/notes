RPC（Remote Procedure Call，远程过程调用）是一种计算机通信协议，允许程序像调用本地方法一样调用另一台计算机（或同一台计算机的不同进程）上的函数或服务，而无需显式处理网络通信细节。

RPC 是[分布式系统](../Distributed%20System/Distributed%20System.md)的基石之一，现代云原生和微服务架构中广泛使用（如 [Kubernetes](../DevOps/Container/Kubernetes.md) 、微服务间的 gRPC 调用）。

### 核心思想

- **透明性**：开发者无需关心网络传输、序列化等底层细节，像调用本地函数一样调用远程服务。
- **跨语言/平台**：通常基于标准协议（如 gRPC、Thrift）实现不同语言和系统间的交互。

### 工作原理

1. **客户端调用**
	- 客户端代码调用一个看似本地的函数（实际是代理/stub（存根））。
2. **参数序列化**
    - 客户端 stub 将参数打包（序列化）成网络可传输格式（如 JSON、[ProtoBuf](../../Language/Data%20Format/ProtoBuf.md)）。
3. **网络传输**
    - 序列化后的数据通过网络发送到服务端（协议如 [HTTP](HTTP.md)、[TCP](网络通讯协议.md#TCP)）。
4. **服务端处理**  
    - 服务端 stub 接收请求，反序列化参数，调用真实的本地函数。
5. **结果返回**
    - 服务端将结果序列化后传回客户端，客户端 stub 反序列化并返回给调用者。


### RPC 框架

- [**gRPC**](../Backend/API%20Design/gRPC.md)：Google 开源的高性能框架，基于 HTTP/2 和 Protocol Buffers。
- **Apache Thrift**：支持多语言的 RPC 框架，由 Facebook 开发。
- **Dubbo**：阿里巴巴开源的 Java RPC 框架，常用于微服务。
- **JSON-RPC**：基于 JSON 的轻量级 RPC 协议。
- [**bRPC**](../Backend/API%20Design/bRPC.md)：百度开源的一款高性能、工业级 RPC 框架。
- [**tRPC**](../Backend/API%20Design/tRPC.md)：一个用于构建类型安全 API 的 TypeScript 框架。



### 优缺点

**优点**

1. **简化开发**：隐藏网络复杂性，提升开发效率。
2. **高性能**：比 REST API 更高效（二进制协议、长连接等）。
3. **强类型**：多数框架支持接口定义语言（IDL），减少错误。


**缺点**

1. **耦合性**：服务端和客户端需共享接口定义，变更可能需同步更新。
2. **调试复杂**：跨进程/网络的问题定位较困难。
3. **依赖网络稳定性**：需处理超时、重试、熔断等问题。