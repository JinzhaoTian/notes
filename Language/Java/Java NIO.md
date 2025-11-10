Java NIO（New Input/Output）是 Java 1.4 引入的一组用于高效 I/O 操作的 API ，提供了与传统 I/O 不同的编程模型。

## 核心组件

1. **通道（Channels）**：连接数据的双向通道
    - `FileChannel`：文件操作
    - `SocketChannel`：TCP 网络通信
    - `ServerSocketChannel`：TCP 服务器端
    - `DatagramChannel`：UDP 通信

2. **缓冲区（Buffers）**：数据容器
    - `ByteBuffer`, `CharBuffer`, `IntBuffer`等
    - 关键属性：capacity, position, limit

3. **选择器(Selectors)**：多路复用器
    - 允许单线程处理多个通道
    - 基于事件驱动模型

## 主要优势

1. [**非阻塞 IO**](../../Computer%20Network/非阻塞%20IO.md) ：通道可以设置为非阻塞模式
2. **内存映射文件**：通过 `FileChannel.map()` 高效处理大文件
3. **分散/聚集**：支持分散读取和聚集写入
4. **选择器机制**：单线程管理多个通道