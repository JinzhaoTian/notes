通过多路复用（Multiplexing）实现高并发是网络编程中的核心技术，其**核心思想**是**用一个进程/线程监听多个 Socket 事件**，避免为每个连接创建独立线程/进程的资源消耗。

## 核心原理

- **事件驱动**：操作系统通知程序哪些 Socket 已就绪（可读/可写/异常），而非阻塞等待单个 Socket。
- **单线程处理多连接**：通过内核机制（如 `select()` / `poll()` / `epoll()` ）批量监控 Socket 状态，减少线程切换开销。


**技术对比**

| 技术       | 性能  | 最大连接数 | 触发方式 | 跨平台性       |
| -------- | --- | ----- | ---- | ---------- |
| `select` | 低   | 1024  | 轮询   | 所有平台       |
| `poll`   | 中   | 无硬限制  | 轮询   | Linux/Unix |
| `epoll`  | 高   | 数十万   | 事件驱动 | Linux      |
| `kqueue` | 高   | 数十万   | 事件驱动 | BSD/macOS  |
|          |     |       |      |            |

**Linux 首选 `epoll` ，Windows 可用 `select` 或 IOCP （异步IO）。**


`select()` / `poll()` / `epoll()` 是内核提供给用户态的多路复用系统调用，线程可以通过一个系统调用函数从内核中获取多个事件。


## 内核机制


### select

`select` 是一种传统的 I/O 多路复用机制，它允许程序监视多个文件描述符，等待一个或多个描述符变为"就绪"状态（即可读、可写或发生异常）。

```c
#include <sys/select.h>

int select(int nfds, fd_set *readfds, fd_set *writefds,
           fd_set *exceptfds, struct timeval *timeout);
```


select 实现多路复用的方式是，将已连接的 socket 都放到一个文件描述符集合，然后调用 select 函数将文件描述符集合拷贝到内核里，让内核来检查是否有网络事件产生，检查的方式很粗暴，就是通过遍历文件描述符集合的方式，当检查到有事件产生后，将此 Socket 标记为可读或可写， 接着再把整个文件描述符集合拷贝回用户态里，然后用户态还需要再通过遍历的方法找到可读或可写的 Socket，然后再对其处理。



#### 使用示例

##### 服务端

```c++
// server.cpp

#include <iostream>
#include <vector>
#include <cstring>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <sys/select.h>
#include <arpa/inet.h>

#define PORT 8080
#define BUFFER_SIZE 1024
#define MAX_CLIENTS 30

int main() {
    int server_fd, new_socket;
    struct sockaddr_in address;
    int opt = 1;
    int addrlen = sizeof(address);
    char buffer[BUFFER_SIZE] = {0};
    
    // 1. 创建主socket
    if (0 == (server_fd = socket(AF_INET, SOCK_STREAM, 0))) {
        perror("socket failed");
        exit(EXIT_FAILURE);
    }
    
    // 2. 设置socket选项
    if (setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR | SO_REUSEPORT, &opt, sizeof(opt))) {
        perror("setsockopt");
        exit(EXIT_FAILURE);
    }
    
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);
    
    // 3. 绑定socket
    if (0 > bind(server_fd, (struct sockaddr *)&address, sizeof(address))) {
        perror("bind failed");
        exit(EXIT_FAILURE);
    }
    
    // 4. 监听
    if (0 > listen(server_fd, MAX_CLIENTS)) {
        perror("listen");
        exit(EXIT_FAILURE);
    }
    
    std::cout << "Server listening on port " << PORT << std::endl;
    
    // 5. 初始化数据结构
    fd_set readfds;
    std::vector<int> client_sockets(MAX_CLIENTS, 0);
    int max_sd, activity;
    
    // 6. 主循环
    while(true) {
        // 清空socket集合
        FD_ZERO(&readfds);
        
        // 添加主socket到集合
        FD_SET(server_fd, &readfds);
        max_sd = server_fd;
        
        // 添加客户端socket到集合
        for (int i = 0; i < MAX_CLIENTS; i++) {
            int sd = client_sockets[i];
            if (sd > 0) {
                FD_SET(sd, &readfds);
            }
            if (sd > max_sd) {
                max_sd = sd;
            }
        }
        
        // 等待活动(阻塞调用)
        activity = select(max_sd + 1, &readfds, NULL, NULL, NULL);
        
        if ((activity < 0) && (errno != EINTR)) {
            std::cerr << "select error" << std::endl;
        }
        
        // 如果是主socket活动，表示有新连接
        if (FD_ISSET(server_fd, &readfds)) {
            if ((new_socket = accept(server_fd, (struct sockaddr *)&address, (socklen_t*)&addrlen)) < 0) {
                perror("accept");
                exit(EXIT_FAILURE);
            }
            
            // 打印新连接信息
            char client_ip[INET_ADDRSTRLEN];
            inet_ntop(AF_INET, &(address.sin_addr), client_ip, INET_ADDRSTRLEN);
            std::cout << "New connection from " << client_ip << ":" << ntohs(address.sin_port) 
                      << ", socket fd: " << new_socket << std::endl;
            
            // 添加新socket到数组
            for (int i = 0; i < MAX_CLIENTS; i++) {
                if (client_sockets[i] == 0) {
                    client_sockets[i] = new_socket;
                    std::cout << "Adding to list of sockets as " << i << std::endl;
                    break;
                }
            }
        }
        
        // 检查客户端socket的IO活动
        for (int i = 0; i < MAX_CLIENTS; i++) {
            int sd = client_sockets[i];
            
            if (FD_ISSET(sd, &readfds)) {
                // 检查是否是断开连接
                int valread = read(sd, buffer, BUFFER_SIZE);
                if (valread == 0) {
                    // 客户端断开连接
                    getpeername(sd, (struct sockaddr*)&address, (socklen_t*)&addrlen);
                    std::cout << "Client disconnected, ip " << inet_ntoa(address.sin_addr) 
                              << ", port " << ntohs(address.sin_port) << std::endl;
                    
                    // 关闭socket并从集合中移除
                    close(sd);
                    client_sockets[i] = 0;
                } else {
                    // 处理客户端消息
                    buffer[valread] = '\0';
                    std::cout << "Received from client " << sd << ": " << buffer << std::endl;
                    
                    // 发送响应
                    const char *response = "Message received by server\n";
                    send(sd, response, strlen(response), 0);
                }
            }
        }
    }
    
    return 0;
}
```

1. **单线程处理多连接**：不需要为每个客户端创建线程
2. **高效**：避免线程上下文切换开销
3. **可扩展**：适合中等数量的并发连接
4. **跨平台**：`select` 在大多数 Unix-like 系统和 Windows 上都可用

### poll



### epoll

epoll 是一种 I/O 事件通知机制，是 Linux 内核实现 I/O 多路复用的一个实现。

**核心数据结构**：
- 1个红黑树
- 1个链表


**核心 API** 

1. **`int epoll_create(int size)`** ：
	- 在 Linux 内核里面申请一个 B+ 树结构文件系统，返回 epoll 对象，也是一个 fd 文件描述符，这个特殊的描述符就是 epoll 实例的句柄，后面的两个接口都以它为中心（即 epfd 形参）。
	- epoll 使用共享内存实现用户空间与内核空间传输数据。

2. **`int epoll_ctl(int epfd， int op， int fd， struct epoll_event *event)`** ：
	- 注册事件，将被监听的描述符添加到红黑树，或从红黑树中删除，或者对监听事件进行修改，具体时间类型可以通过 op 参数传入。
    - op 参数

3. **`int epoll_wait(int epfd， struct epoll_event *events， int maxevents， int timeout)`** ：
	- 阻塞，等待注册的事件发生，返回事件的数目，并将触发的事件写入 events 数组中。 处于 ready 状态的那些文件描述符会被复制进 ready list 中，`epoll_wait` 用于向用户进程返回 ready list 。
	- events 和 maxevents 两个参数描述一个由用户分配的 struct epoll event 数组，调用返回时，内核将 ready list 复制到这个数组中，并将实际复制的个数作为返回值。
	- 注意，如果 ready list 比 maxevents 长，则只能复制前 maxevents 个成员；反之，则能够完全复制 ready list 。

**触发方式**

epoll的触发机制有两个：水平触发和边缘触发，默认是水平触发。

- **水平触发（LT）**：对于读操作，只要缓冲内容不为空，LT模式返回读就绪。对于写操作，只要缓冲区还不满，LT模式会返回写就绪。也就是说只要可操作，就一直会触发。
- **边缘触发（ET）**：只有从不可操作，转变成可操作的时候才会触发。该模式在很大程度上减少了epoll事件被重复触发的次数，因此效率要比LT模式高。epoll工作在ET模式的时候，必须使用非阻塞套接口，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。



**epoll与select的对比**

1. select：描述符有个数限制，默认1024。采用轮询方式全盘扫描。每次调用 select() 需要把 fd 集合从用户态拷贝到内核态，并进行遍历。
2. epoll：描述符无个数限制。效率提高，使用回调通知而不是轮询的方式。内核和用户空间mmap同一块内存实现(mmap是一种内存映射文件的方法，即将一个文件或者其它对象映射到进程的地址空间)

epoll 引入了新的数据结构，带来了复杂性，如果并发低，比如低于1024，并且每个socket都处于一直活跃的状态，那么性能反而不如select/poll。


#### 使用示例

##### 服务端

```c++
// server.cpp

#include <iostream>
#include <vector>
#include <cstring>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <sys/epoll.h>
#include <arpa/inet.h>

#define PORT 8080
#define BUFFER_SIZE 1024
#define MAX_EVENTS 64
#define MAX_CLIENTS 1000

int main() {
    int server_fd, new_socket;
    struct sockaddr_in address;
    int opt = 1;
    int addrlen = sizeof(address);
    char buffer[BUFFER_SIZE] = {0};
    
    // 1. 创建主socket
    if (0 == (server_fd = socket(AF_INET, SOCK_STREAM | SOCK_NONBLOCK, 0))) {
        perror("socket failed");
        exit(EXIT_FAILURE);
    }
    
    // 2. 设置socket选项
    if (setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR | SO_REUSEPORT, &opt, sizeof(opt))) {
        perror("setsockopt");
        exit(EXIT_FAILURE);
    }
    
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);
    
    // 3. 绑定socket
    if (0 > bind(server_fd, (struct sockaddr *)&address, sizeof(address))) {
        perror("bind failed");
        exit(EXIT_FAILURE);
    }
    
    // 4. 监听
    if (0 > listen(server_fd, SOMAXCONN)) {
        perror("listen");
        exit(EXIT_FAILURE);
    }
    
    std::cout << "Server listening on port " << PORT << std::endl;
    
    // 5. 创建epoll实例
    int epoll_fd = epoll_create1(0);
    if (-1 == epoll_fd) {
        perror("epoll_create1");
        exit(EXIT_FAILURE);
    }
    
    // 6. 添加服务器socket到epoll
    struct epoll_event event;
    event.events = EPOLLIN | EPOLLET; // 边缘触发模式
    event.data.fd = server_fd;
    if (-1 == epoll_ctl(epoll_fd, EPOLL_CTL_ADD, server_fd, &event)) {
        perror("epoll_ctl: server_fd");
        exit(EXIT_FAILURE);
    }
    
    // 7. 事件缓冲区
    struct epoll_event events[MAX_EVENTS];
    
    // 8. 主循环
    while(true) {
        int nfds = epoll_wait(epoll_fd, events, MAX_EVENTS, -1);
        if (-1 == nfds) {
            perror("epoll_wait");
            exit(EXIT_FAILURE);
        }
        
        for (int i = 0; i < nfds; i++) {
            // 处理新连接
            if (events[i].data.fd == server_fd) {
                while(true) {
                    new_socket = accept4(server_fd, (struct sockaddr *)&address, 
                                       (socklen_t*)&addrlen, SOCK_NONBLOCK);
                    if (new_socket == -1) {
                        if (errno == EAGAIN || errno == EWOULDBLOCK) {
                            // 已经处理完所有新连接
                            break;
                        } else {
                            perror("accept");
                            break;
                        }
                    }
                    
                    // 打印新连接信息
                    char client_ip[INET_ADDRSTRLEN];
                    inet_ntop(AF_INET, &(address.sin_addr), client_ip, INET_ADDRSTRLEN);
                    std::cout << "New connection from " << client_ip << ":" 
                              << ntohs(address.sin_port) << ", socket fd: " << new_socket << std::endl;
                    
                    // 添加新socket到epoll
                    event.events = EPOLLIN | EPOLLET | EPOLLRDHUP;
                    event.data.fd = new_socket;
                    if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, new_socket, &event) == -1) {
                        perror("epoll_ctl: new_socket");
                        close(new_socket);
                    }
                }
            } 
            // 处理客户端数据或断开连接
            else {
                int client_fd = events[i].data.fd;
                
                // 检查连接是否关闭
                if (events[i].events & (EPOLLRDHUP | EPOLLHUP)) {
                    std::cout << "Client disconnected, socket fd: " << client_fd << std::endl;
                    close(client_fd);
                    continue;
                }
                
                // 处理可读事件
                if (events[i].events & EPOLLIN) {
                    while(true) {
                        ssize_t count = read(client_fd, buffer, BUFFER_SIZE - 1);
                        if (count == -1) {
                            if (errno != EAGAIN) {
                                perror("read");
                                close(client_fd);
                            }
                            break;
                        } else if (count == 0) {
                            // 客户端正常关闭连接
                            std::cout << "Client closed connection, socket fd: " << client_fd << std::endl;
                            close(client_fd);
                            break;
                        } else {
                            // 处理接收到的数据
                            buffer[count] = '\0';
                            std::cout << "Received from client " << client_fd << ": " << buffer << std::endl;
                            
                            // 发送响应
                            const char *response = "Message received by server\n";
                            if (write(client_fd, response, strlen(response)) == -1) {
                                perror("write");
                                close(client_fd);
                                break;
                            }
                        }
                    }
                }
            }
        }
    }
    
    close(server_fd);
    close(epoll_fd);
    return 0;
}
```

### IOCP

IOCP（Input/Output Completion Ports，输入/输出完成端口）是 Windows 操作系统提供的一种高效 I/O 模型，主要用于处理大量并发异步 I/O 操作。

IOCP 是 Windows 平台上最高效的 I/O 模型，它：
- 允许应用程序异步处理多个 I/O 操作
- 使用线程池高效管理 I/O 完成通知
- 特别适合高并发网络服务器应用

**工作原理**

1. **创建完成端口**：首先创建一个完成端口对象
2. **关联文件句柄**：将需要进行异步 I/O 的文件/套接字句柄与完成端口关联
3. **工作线程池**：创建多个工作线程等待完成端口的通知
4. **I/O 操作**：发起异步 I/O 操作
5. **完成通知**：当 I/O 操作完成时，系统将完成通知放入完成端口队列
6. **线程处理**：工作线程从队列中获取完成通知并处理

**主要特点**

- **可伸缩性**：能有效利用多核 CPU，性能随处理器数量线性增长
- **高效线程管理**：减少线程上下文切换，避免"惊群"问题
- **负载均衡**：系统自动将 I/O 完成通知分配给空闲线程
- **支持多种 I/O 类型**：文件、套接字、命名管道等

**使用场景**

- 高性能网络服务器(Web服务器、游戏服务器等)
- 需要处理大量并发连接的应用程序
- 需要高吞吐量 I/O 操作的应用

相关 API 函数

- `CreateIoCompletionPort` - 创建或关联完成端口
- `GetQueuedCompletionStatus` - 获取完成状态
- `PostQueuedCompletionStatus` - 向完成端口投递自定义通知

IOCP 是 Windows 平台开发高性能服务器应用程序的重要技术，相比 select、WSAAsyncSelect 等模型，它能提供更好的性能和可伸缩性。


### kqueue