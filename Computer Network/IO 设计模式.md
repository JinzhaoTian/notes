
I/O 设计模式的核心是解决 Socket 的**高效管理**问题，尤其是应对海量连接时的性能瓶颈。

[参考](https://xiaolincoding.com/os/8_network_system/reactor.html)

**Reactor 是非阻塞同步网络模式**，而 **Proactor 是异步网络模式**。

## Reactor

事件驱动模型（Reactor 模式）是一种**高性能的 I/O 设计模式**，用于处理并发请求，尤其适合网络服务场景（如 Web 服务器、消息中间件等）。

其核心思想是**通过事件分发机制，用单线程或少量的线程处理大量I/O操作**，避免为每个请求创建线程的开销。

### 核心原理

1. **事件循环（Event Loop）**
	- 主线程不断监听并分发事件（如网络连接、数据到达等），事件触发后调用对应的回调函数。
2. **非阻塞 I/O** 
	- 所有I/O操作设置为非阻塞模式，通过系统调用（如 `epoll`、`kqueue` ）监听 I/O 就绪状态，避免线程空等。
3. **回调机制（Callback）**
	- 预先注册事件处理函数，事件发生时自动触发回调，无需主动轮询。


### 关键组件

1. **Reactor**
	- 负责监听和分发事件（如新连接、数据可读/可写）。
2. **Handlers**
	- 处理具体事件的回调函数（如处理请求、返回响应）。
3. **Demultiplexer（多路复用器）**
	- 底层系统调用（如`select`/`epoll`）监听多个I/O源，通知就绪事件。

### 工作流程

1. **注册事件**
	 - 将需要监听的I/O事件（如`socket.read`）注册到Reactor。
2. **事件监听**
	- Reactor通过多路复用器（如`epoll`）监听所有事件。
3. **事件触发**
	- 当某个I/O就绪（如数据到达），多路复用器返回就绪事件。
4. **分发事件**
	- Reactor将事件分发给对应的Handler处理。
5. **回调执行**
	- Handler处理完成后，可能再次注册新事件（如等待下一次请求）。


### 应用场景

- [**Nginx**](../Backend/架构设计/Nginx.md)：高性能 Web 服务器。
- [**Redis**](../Database/Redis/基础架构.md)：单线程 Reactor 处理客户端请求。
- **Node.js**：事件驱动型 JavaScript 运行时。


### 简单实现

#### C++

```c++
// reactor_server.h

#ifndef REACTOR_SERVER_H
#define REACTOR_SERVER_H

#include <sys/epoll.h>
#include <functional>
#include <unordered_map>
#include <memory>

class Reactor {
public:
    using EventHandler = std::function<void(int)>;
    
    Reactor();
    ~Reactor();
    
    void register_handler(int fd, uint32_t events, const EventHandler& handler);
    void remove_handler(int fd);
    void run();
    
private:
    int epoll_fd;
    std::unordered_map<int, EventHandler> handlers;
};

class TCPServer {
public:
    TCPServer(int port);
    ~TCPServer();
    
    void start();
    
private:
    void handle_accept();
    void handle_read(int client_fd);
    
    int server_fd;
    int port;
    std::unique_ptr<Reactor> reactor;
};

#endif // REACTOR_SERVER_H
```


```c++
// reactor_server.cpp

#include "reactor_server.h"
#include <iostream>
#include <cstring>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define MAX_EVENTS 64
#define BUFFER_SIZE 1024

// Reactor 实现
Reactor::Reactor() {
    epoll_fd = epoll_create1(0);
    if (epoll_fd == -1) {
        throw std::runtime_error("Failed to create epoll instance");
    }
}

Reactor::~Reactor() {
    close(epoll_fd);
}

void Reactor::register_handler(int fd, uint32_t events, const EventHandler& handler) {
    struct epoll_event event;
    event.events = events;
    event.data.fd = fd;
    
    if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, fd, &event) == -1) {
        throw std::runtime_error("Failed to add fd to epoll");
    }
    
    handlers[fd] = handler;
}

void Reactor::remove_handler(int fd) {
    if (epoll_ctl(epoll_fd, EPOLL_CTL_DEL, fd, nullptr) == -1) {
        std::cerr << "Failed to remove fd from epoll" << std::endl;
    }
    
    handlers.erase(fd);
    close(fd);
}

void Reactor::run() {
    struct epoll_event events[MAX_EVENTS];
    
    while (true) {
        int nfds = epoll_wait(epoll_fd, events, MAX_EVENTS, -1);
        if (nfds == -1) {
            throw std::runtime_error("epoll_wait failed");
        }
        
        for (int i = 0; i < nfds; ++i) {
            int fd = events[i].data.fd;
            auto it = handlers.find(fd);
            if (it != handlers.end()) {
                it->second(fd);
            }
        }
    }
}

// TCPServer 实现
TCPServer::TCPServer(int port) : port(port), reactor(std::make_unique<Reactor>()) {
    server_fd = socket(AF_INET, SOCK_STREAM | SOCK_NONBLOCK, 0);
    if (server_fd == -1) {
        throw std::runtime_error("Failed to create server socket");
    }
    
    int opt = 1;
    if (setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR | SO_REUSEPORT, &opt, sizeof(opt))) {
        throw std::runtime_error("Failed to set socket options");
    }
    
    sockaddr_in address;
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(port);
    
    if (bind(server_fd, (sockaddr*)&address, sizeof(address)) < 0) {
        throw std::runtime_error("Failed to bind socket");
    }
    
    if (listen(server_fd, SOMAXCONN) < 0) {
        throw std::runtime_error("Failed to listen on socket");
    }
}

TCPServer::~TCPServer() {
    if (server_fd != -1) {
        close(server_fd);
    }
}

void TCPServer::handle_accept() {
    sockaddr_in client_addr;
    socklen_t client_len = sizeof(client_addr);
    
    while (true) {
        int client_fd = accept4(server_fd, (sockaddr*)&client_addr, &client_len, SOCK_NONBLOCK);
        if (client_fd == -1) {
            if (errno == EAGAIN || errno == EWOULDBLOCK) {
                break;
            }
            std::cerr << "Failed to accept client" << std::endl;
            continue;
        }
        
        char client_ip[INET_ADDRSTRLEN];
        inet_ntop(AF_INET, &client_addr.sin_addr, client_ip, INET_ADDRSTRLEN);
        std::cout << "New connection from " << client_ip << ":" << ntohs(client_addr.sin_port) << std::endl;
        
        reactor->register_handler(client_fd, EPOLLIN | EPOLLRDHUP, 
            [this](int fd) { this->handle_read(fd); });
    }
}

void TCPServer::handle_read(int client_fd) {
    char buffer[BUFFER_SIZE];
    
    while (true) {
        ssize_t count = read(client_fd, buffer, BUFFER_SIZE - 1);
        if (count == -1) {
            if (errno != EAGAIN) {
                std::cerr << "Error reading from client" << std::endl;
                reactor->remove_handler(client_fd);
            }
            break;
        } else if (count == 0) {
            std::cout << "Client disconnected" << std::endl;
            reactor->remove_handler(client_fd);
            break;
        } else {
            buffer[count] = '\0';
            std::cout << "Received: " << buffer;
            
            const char* response = "Message received\n";
            if (write(client_fd, response, strlen(response)) == -1) {
                std::cerr << "Failed to send response" << std::endl;
                reactor->remove_handler(client_fd);
                break;
            }
        }
    }
}

void TCPServer::start() {
    reactor->register_handler(server_fd, EPOLLIN, 
        [this](int) { this->handle_accept(); });
    
    std::cout << "Server started on port " << port << std::endl;
    reactor->run();
}
```


```C++
// main.cpp

#include "reactor_server.h"
#include <iostream>

int main() {
    try {
        TCPServer server(8080);
        server.start();
    } catch (const std::exception& e) {
        std::cerr << "Error: " << e.what() << std::endl;
        return 1;
    }
    
    return 0;
}
```

**核心组件**

1. **Initiation Dispatcher (Reactor 类)**:
    - 核心事件循环
    - 使用 epoll 进行事件多路复用
    - 管理事件处理器注册与删除
2. **Event Handler (函数对象)**:
    - 处理特定类型的事件
    - 通过 std::function 实现回调
3. **Concrete Event Handler (TCPServer 类)**:
    - 处理具体的网络事件
    - 包括接受连接和处理数据



## Proactor

**Proactor**（Proactive Reactor）是一种**基于异步 I/O 的高性能事件处理模式**，与**Reactor**（基于同步非阻塞I/O）不同，Proactor **由操作系统内核完成 I/O 操作**，应用程序只需处理就绪事件，无需主动读写数据。

### 核心原理

1. **异步I/O（Asynchronous I/O）**
    - 应用程序发起I/O请求（如 `aio_read` 、 `aio_write` ）后立即返回，**内核负责完成实际读写**，并在操作完成后通知应用程序。
    - 对比 Reactor（需应用程序自己读写数据）。

2. **事件完成通知**
    - 内核完成 I/O 后，通过**回调**或**事件队列**通知应用程序（如 `IOCP` 在 Windows ，`io_uring` 在 Linux ）。

3. **分离"发起请求"和"处理结果"**
    - 应用程序只管发起 I/O 请求，内核负责执行并返回结果。

### 关键组件

1. **Proactor**
    - 负责协调异步 I/O 操作，管理事件完成队列。

2. **Completion Handler（完成处理器）**
    - 处理 I/O 完成事件的回调函数（如解析接收到的数据）。

3. **Asynchronous Operation Processor（异步操作处理器）**
    - 底层系统（如 `IOCP` 、 `io_uring` ）执行实际 I/O ，完成后通知 Proactor 。


### 工作流程

1. **发起异步 I/O 请求**
    - 应用程序调用 `aio_read`（异步读），内核接管后续操作。

2. **内核执行 I/O **
    - 数据就绪后，内核将数据直接写入用户提供的缓冲区。

3. **通知完成**
    - 内核通过事件队列（如 `IOCP` ）或信号（如 `SIGIO` ）通知应用程序。

4. **回调处理**
    - Proactor 调用预先注册的 `Completion Handler` 处理数据。


### 应用场景

- **Windows IOCP**：高性能服务器（如游戏后端）。
- **Linux io_uring**：新一代异步 I/O 框架（如 Redis 6.0+ 可选支持）。
- **数据库连接池**：异步执行 SQL 查询。

### 简单实现

#### C++

```C++
// proactor_server.h

#ifndef PROACTOR_SERVER_H
#define PROACTOR_SERVER_H

#include <liburing.h>
#include <functional>
#include <unordered_map>
#include <memory>
#include <vector>

class Proactor {
public:
    using CompletionHandler = std::function<void(int, int)>;
    
    Proactor(unsigned entries = 128);
    ~Proactor();
    
    void submit_accept(int fd, sockaddr* addr, socklen_t* addrlen, void* user_data);
    void submit_read(int fd, void* buf, size_t count, void* user_data);
    void submit_write(int fd, const void* buf, size_t count, void* user_data);
    void run(const std::function<void(int, int)>& completion_handler);
    
private:
    struct io_uring ring;
    std::unordered_map<uint64_t, int> user_data_to_fd;
    uint64_t next_user_data = 1;
};

class TCPServer {
public:
    TCPServer(int port);
    ~TCPServer();
    
    void start();
    
private:
    void handle_accept(int res, int user_data);
    void handle_read(int res, int client_fd);
    void handle_write(int res, int client_fd);
    void start_accept();
    
    int server_fd;
    int port;
    std::unique_ptr<Proactor> proactor;
    std::vector<char> buffer;
};

#endif // PROACTOR_SERVER_H
```


```c++
// proactor_server.cpp

#include "proactor_server.h"
#include <iostream>
#include <cstring>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define BUFFER_SIZE 1024
#define BACKLOG 128

// Proactor 实现
Proactor::Proactor(unsigned entries) {
    if (io_uring_queue_init(entries, &ring, 0) < 0) {
        throw std::runtime_error("Failed to initialize io_uring");
    }
}

Proactor::~Proactor() {
    io_uring_queue_exit(&ring);
}

void Proactor::submit_accept(int fd, sockaddr* addr, socklen_t* addrlen, void* user_data) {
    auto* sqe = io_uring_get_sqe(&ring);
    if (!sqe) {
        throw std::runtime_error("Failed to get submission queue entry");
    }
    
    io_uring_prep_accept(sqe, fd, addr, addrlen, 0);
    io_uring_sqe_set_data(sqe, user_data);
    
    io_uring_submit(&ring);
}

void Proactor::submit_read(int fd, void* buf, size_t count, void* user_data) {
    auto* sqe = io_uring_get_sqe(&ring);
    if (!sqe) {
        throw std::runtime_error("Failed to get submission queue entry");
    }
    
    io_uring_prep_read(sqe, fd, buf, count, 0);
    io_uring_sqe_set_data(sqe, user_data);
    
    io_uring_submit(&ring);
}

void Proactor::submit_write(int fd, const void* buf, size_t count, void* user_data) {
    auto* sqe = io_uring_get_sqe(&ring);
    if (!sqe) {
        throw std::runtime_error("Failed to get submission queue entry");
    }
    
    io_uring_prep_write(sqe, fd, buf, count, 0);
    io_uring_sqe_set_data(sqe, user_data);
    
    io_uring_submit(&ring);
}

void Proactor::run(const std::function<void(int, int)>& completion_handler) {
    while (true) {
        io_uring_cqe* cqe;
        int ret = io_uring_wait_cqe(&ring, &cqe);
        if (ret < 0) {
            throw std::runtime_error("io_uring_wait_cqe failed");
        }
        
        void* user_data = io_uring_cqe_get_data(cqe);
        int res = cqe->res;
        
        if (user_data) {
            completion_handler(res, reinterpret_cast<uint64_t>(user_data));
        }
        
        io_uring_cqe_seen(&ring, cqe);
    }
}

// TCPServer 实现
TCPServer::TCPServer(int port) : port(port), buffer(BUFFER_SIZE), 
    proactor(std::make_unique<Proactor>()) {
    server_fd = socket(AF_INET, SOCK_STREAM | SOCK_NONBLOCK, 0);
    if (server_fd == -1) {
        throw std::runtime_error("Failed to create server socket");
    }
    
    int opt = 1;
    if (setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR | SO_REUSEPORT, &opt, sizeof(opt))) {
        throw std::runtime_error("Failed to set socket options");
    }
    
    sockaddr_in address;
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(port);
    
    if (bind(server_fd, (sockaddr*)&address, sizeof(address)) < 0) {
        throw std::runtime_error("Failed to bind socket");
    }
    
    if (listen(server_fd, BACKLOG) < 0) {
        throw std::runtime_error("Failed to listen on socket");
    }
}

TCPServer::~TCPServer() {
    if (server_fd != -1) {
        close(server_fd);
    }
}

void TCPServer::start_accept() {
    sockaddr_in client_addr;
    socklen_t client_len = sizeof(client_addr);
    
    // 使用 user_data 为0表示是accept操作
    proactor->submit_accept(server_fd, (sockaddr*)&client_addr, &client_len, 
                          reinterpret_cast<void*>(0));
}

void TCPServer::handle_accept(int res, int user_data) {
    if (res < 0) {
        std::cerr << "Accept failed: " << strerror(-res) << std::endl;
        start_accept();
        return;
    }
    
    int client_fd = res;
    char client_ip[INET_ADDRSTRLEN];
    sockaddr_in client_addr;
    socklen_t client_len = sizeof(client_addr);
    
    if (getpeername(client_fd, (sockaddr*)&client_addr, &client_len) == 0) {
        inet_ntop(AF_INET, &client_addr.sin_addr, client_ip, INET_ADDRSTRLEN);
        std::cout << "New connection from " << client_ip << ":" 
                  << ntohs(client_addr.sin_port) << std::endl;
    }
    
    // 提交读请求
    proactor->submit_read(client_fd, buffer.data(), buffer.size(), 
                         reinterpret_cast<void*>(client_fd));
    
    // 继续接受新连接
    start_accept();
}

void TCPServer::handle_read(int res, int client_fd) {
    if (res <= 0) {
        if (res < 0) {
            std::cerr << "Read error: " << strerror(-res) << std::endl;
        }
        std::cout << "Client disconnected" << std::endl;
        close(client_fd);
        return;
    }
    
    std::cout << "Received " << res << " bytes from client " << client_fd << ": "
              << std::string(buffer.data(), res) << std::endl;
    
    // 回显数据
    proactor->submit_write(client_fd, buffer.data(), res, 
                          reinterpret_cast<void*>(client_fd));
}

void TCPServer::handle_write(int res, int client_fd) {
    if (res < 0) {
        std::cerr << "Write error: " << strerror(-res) << std::endl;
        close(client_fd);
        return;
    }
    
    // 继续读取数据
    proactor->submit_read(client_fd, buffer.data(), buffer.size(), 
                         reinterpret_cast<void*>(client_fd));
}

void TCPServer::start() {
    start_accept();
    
    auto handler = [this](int res, int user_data) {
        if (user_data == 0) {
            handle_accept(res, user_data);
        } else {
            int client_fd = user_data;
            if (res > 0) {
                // 判断是读完成还是写完成
                // 这里简化处理，实际应该根据操作类型区分
                if (true) { // 这里应该有更好的方式判断操作类型
                    handle_read(res, client_fd);
                } else {
                    handle_write(res, client_fd);
                }
            } else {
                handle_read(res, client_fd);
            }
        }
    };
    
    std::cout << "Server started on port " << port << std::endl;
    proactor->run(handler);
}
```


```c++
// main.cpp

#include "proactor_server.h"
#include <iostream>

int main() {
    try {
        TCPServer server(8080);
        server.start();
    } catch (const std::exception& e) {
        std::cerr << "Error: " << e.what() << std::endl;
        return 1;
    }
    
    return 0;
}
```

**核心组件**：

1. **Asynchronous Operation Processor (Proactor 类)**:
    - 使用 `io_uring` 提交异步 I/O 操作
    - 管理操作完成队列
2. **Completion Handler (函数对象)**:
    - 处理已完成的异步操作
    - 通过 std::function 实现回调
3. **Concrete Completion Handler (TCPServer 类)**:
    - 处理具体的网络事件完成
    - 包括接受连接、读取数据和写入数据