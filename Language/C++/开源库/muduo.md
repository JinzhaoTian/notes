muduo 是一个用 C++ 开发的高性能网络库，专注于构建多线程网络服务器，适合对高性能和低延迟要求较高的场景。它由陈硕（Shuo Chen）开发，最初作为开源项目发布，主要用于学习和研究现代高性能服务器编程的最佳实践。

### 主要特点

1. **高性能**
    - 基于事件驱动（event-driven）的异步 I/O 模型。
    - 使用 **Reactor** 模式，充分利用 Linux 的 epoll 或 kqueue 等高效的 I/O 多路复用机制。
2. **多线程**
    - 提供高效的线程池管理，允许任务在线程池中分发和执行。
    - 能够很好地利用多核 CPU 的性能。
3. **易用性**
    - 提供直观的接口，用于创建 TCP 客户端和服务器。
    - 简化了多线程编程中的复杂性，如线程安全和资源同步。
4. **稳定性**
    - 支持大规模连接（如百万连接）。
    - 提供内建的日志系统和错误处理机制。
5. **现代 C++ 风格**
    - 使用 RAII（资源获取即初始化）和智能指针，减少内存管理的复杂性。
    - 遵循 C++ 标准的设计模式和最佳实践。



### 主要模块

1. **EventLoop**
    - 核心模块，负责管理事件循环，监听文件描述符上的事件，并调用相应的回调函数。
2. **Channel**
    - 抽象了一个文件描述符和其感兴趣的事件（如读、写、关闭），并绑定回调函数。
3. **Poller**
    - 提供 epoll 或 kqueue 的具体实现，用于事件的高效分发。
4. **TcpServer 和 TcpClient**
    - 提供构建 TCP 服务器和客户端的接口，支持高效的连接管理和数据收发。
5. **ThreadPool**
    - 实现了线程池，支持将任务分配到线程池中执行，提高多线程编程的效率。



```c++
#include <muduo/net/TcpServer.h>
#include <muduo/net/EventLoop.h>
#include <muduo/base/Logging.h>

using namespace muduo;
using namespace muduo::net;

void onMessage(const TcpConnectionPtr& conn, Buffer* buf, Timestamp time) {
    std::string msg = buf->retrieveAllAsString();
    LOG_INFO << "Received: " << msg;
    conn->send(msg); // 回显消息
}

int main() {
    EventLoop loop;
    InetAddress listenAddr(8080); // 监听 8080 端口
    TcpServer server(&loop, listenAddr, "EchoServer");

    server.setMessageCallback(onMessage);
    server.start(); // 开始监听
    loop.loop();    // 启动事件循环
}
```