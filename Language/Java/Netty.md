Netty 是一个**高性能、异步事件驱动**的 Java 网络应用框架，主要用于快速开发高可靠、高可扩展的网络服务器和客户端程序。

基于 NIO（Non-blocking I/O，非阻塞 I/O）技术，优化了 TCP/UDP、HTTP 等协议的通信，广泛应用于分布式系统、游戏服务器、RPC 框架（如 Dubbo）、消息中间件（如 [RocketMQ](../../Backend/消息队列/RocketMQ.md) ）等领域。

### 核心特点

1. **异步非阻塞 I/O**
    - 基于 [Java NIO](Java%20NIO.md)，使用事件驱动模型（ [Reactor 模式](../../Computer%20Network/IO%20设计模式.md#Reactor) ），避免线程阻塞，支持高并发连接。
    - 通过 `ChannelFuture` 实现异步操作，回调机制通知结果。

2. **高性能**
    - [零拷贝技术（Zero-Copy）](../../Computer%20Network/零拷贝技术.md)：减少内存拷贝，提升数据传输效率。
    - 内存池优化：重用缓冲区（ByteBuf），降低 GC 压力。

3. **模块化设计**
    - 提供可扩展的编解码器（Codec）、处理器（Handler）等组件，支持自定义协议（如 [HTTP](../../Computer%20Network/HTTP.md)、[WebSocket](../../Computer%20Network/WebSocket.md)、[ProtoBuf](../Data%20Format/ProtoBuf.md) 等）。

4. **健壮性**
    - 自动处理网络断连、流量控制、异常等场景。

5. **简洁的 API**
    - 相比原生 Java NIO，Netty 的 API 更易用，隐藏了底层复杂性（如 Selector、ByteBuffer 等）。