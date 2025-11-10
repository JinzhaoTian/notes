**libuv** 是一个高性能、跨平台的异步 I/O 库，主要用于 Node.js，但也广泛应用于其他系统软件。它封装了不同操作系统（Windows、Linux、macOS 等）的底层 I/O 操作，提供统一的 API 支持事件驱动编程。

## 核心功能

1. **事件循环（Event Loop）**
    - libuv 的核心是[事件循环](../../Operation%20System/异步编程.md#事件循环)，负责处理异步任务（如 I/O、定时器、网络请求等）。
    - 通过轮询机制（ [epoll](../IO%20多路复用.md#epoll) / [kqueue](../IO%20多路复用.md#kqueue) / [IOCP](../IO%20多路复用.md#IOCP) ）高效调度事件。

2. **跨平台 I/O 抽象**
    - 统一文件系统操作、网络（ TCP / UDP ）、进程/线程管理、管道等。
    - 在 Windows 使用 IOCP，在 Linux/macOS 使用 epoll / kqueue。

3. **异步操作支持**
    - 文件读写（`uv_fs_*`）、DNS 解析（`uv_getaddrinfo`）、子进程（`uv_spawn`）等。
    - 非阻塞的 TCP/UDP 通信（`uv_tcp_t`/`uv_udp_t`）。

4. **其他工具**
    - 定时器（`uv_timer_t`）、线程池（默认 4 个线程）、信号处理（`uv_signal_t`）。


### 事件循环

![](../../Operation%20System/_imgs/Pasted%20image%2020250623164453.png)

libuv 的[事件循环（Event Loop）](../../Operation%20System/异步编程.md#事件循环)是一个**单线程循环**，基于 [Reactor](../IO%20设计模式.md#Reactor) 模式，并采用 [IO 多路复用](../IO%20多路复用.md)技术来监听文件描述符（File Descriptors，fd）、定时器（Timers）和任务队列（Callbacks）等 I/O 事件，并按优先级执行。


#### 基本结构

- **`uv_loop_t`**：代表事件循环本身，存储所有待处理的事件和回调
```c
uv_loop_t *loop = uv_default_loop();  // 获取默认事件循环
uv_run(loop, UV_RUN_DEFAULT);         // 启动事件循环
```


- **`uv_handle_t`**：各种 I/O 操作的抽象基类（如 TCP、UDP、文件操作）
```c
uv_tcp_t tcp_handle;
uv_tcp_init(loop, &tcp_handle); // 初始化 TCP 句柄
```


- **`uv_req_t`**：异步请求基类（如文件读写、DNS 查询）
```c
uv_fs_t fs_req;
uv_fs_read(loop, &fs_req, fd, buffer, length, offset, callback); // 异步文件读取
```

```
           ┌───────────────┐
           │   uv_loop_t   │
           └──────┬───────┘
                  │
     ┌────────────┼────────────┐
     ▼            ▼            ▼
┌─────────┐  ┌─────────┐  ┌─────────┐
│uv_handle│  │ uv_req  │  │uv_timer │
└────┬────┘  └────┬────┘  └─────────┘
     │            │
     ▼            ▼
┌─────────┐  ┌─────────┐
│uv_tcp_t │  │ uv_fs_t │
└─────────┘  └─────────┘
```



#### 核心机制

```
   ┌───────────────────────┐
   │        Timers         │ → 检查 setTimeout / setInterval
   └──────────┬────────────┘
   ┌──────────▼────────────┐
   │     Pending I/O       │ → 执行挂起的系统级回调（如 TCP 错误）
   └──────────┬────────────┘
   ┌──────────▼────────────┐
   │        Polling        │ → 等待新 I/O 事件（核心阶段）
   │   (epoll/kqueue/IOCP) │
   └──────────┬────────────┘
   ┌──────────▼────────────┐
   │        Check          │ → 执行 setImmediate 回调
   └──────────┬────────────┘
   ┌──────────▼────────────┐
   │    Close Callbacks    │ → 执行 'close' 事件回调（如 socket.destroy()）
   └──────────────────────┘
```

1. **I/O 多路复用**（I/O Polling）：使用操作系统提供的 I/O 多路复用机制来监听多个文件描述符
	- Linux → [epoll](../IO%20多路复用.md#epoll)
	- macOS/BSD → [kqueue](../IO%20多路复用.md#kqueue)
	- Windows → [IOCP](../IO%20多路复用.md#IOCP)

2. **任务队列**（Callback Queue）：事件循环维护多个任务队列，按以下优先级处理回调：
	- Timer Queue（定时器队列）
	- Pending Queue（待处理 I/O 回调队列）
	- Idle/Prepare Queue（闲置/准备队列）：内部使用的阶段，通常不涉及用户代码。
	- Poll Queue（轮询队列）
	- Check Queue（检查队列）
	- Close Queue（关闭回调队列）


3. **线程池**（Thread Pool）：由于文件 I/O 和 CPU 密集型任务可能阻塞事件循环，libuv 使用线程池（默认 4 个线程）来执行它们


#### 实现代码

1. 基本循环：
```c
#include <uv.h>

int main() {
    uv_loop_t *loop = uv_default_loop();

    // 添加一个定时器
    uv_timer_t timer;
    uv_timer_init(loop, &timer);
    uv_timer_start(&timer, [](uv_timer_t *handle) {
        printf("Timer fired!\n");
    }, 1000, 0);  // 1秒后触发

    uv_run(loop, UV_RUN_DEFAULT);  // 运行事件循环
    return 0;
}
```

2. 文件 I/O（使用线程池）：
```c
uv_fs_t req;
uv_fs_read(loop, &req, file_fd, buffer, length, offset, [](uv_fs_t *req) {
    if (req->result < 0) {
        fprintf(stderr, "Read error: %s\n", uv_strerror(req->result));
    } else {
        printf("Read %zd bytes\n", req->result);
    }
    uv_fs_req_cleanup(req);  // 清理请求
});
```

