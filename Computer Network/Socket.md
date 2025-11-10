Socket（套接字）是计算机网络通信的核心概念之一，它充当不同主机或同一主机上不同进程间双向通信的端点。

**核心功能**

- **建立连接**（如 `bind()`、`listen()`、`accept()`）。
- **数据传输**（如 `send()`、`recv()`）。
- **设置阻塞/非阻塞模式**（如 `fcntl(O_NONBLOCK)`）。

**工作模式**

- **阻塞 Socket** ：`recv()` 会阻塞线程直到数据到达
- **非阻塞 Socket** ：`recv()` 立即返回，需轮询或事件通知


### 核心概念

- **通信端点**：
	- Socket 是 IP 地址 + 端口号的组合，唯一标识网络中的一个进程（类似电话号码 + 分机号）。
- **抽象层**：
	- 操作系统提供的 API ，隐藏底层协议（如 TCP / IP ）的复杂性，允许开发者通过读写接口实现网络通信。


### 类型

1. **流式 Socket（SOCK_STREAM）**：
    - 基于TCP协议，提供可靠、面向连接的字节流服务。
    - 特点：数据无丢失、按序到达（如 HTTP 、 FTP ）。

2. **数据报 Socket（SOCK_DGRAM）**：
    - 基于UDP协议，无连接、不可靠，但效率高（如视频流、DNS查询）。

- **原始 Socket（SOCK_RAW）**：
    - 直接访问底层协议（如 ICMP ），用于开发自定义协议或网络工具（如 ping ）。


### 操作流程

以 TCP 为例：

| **服务端**                      | **客户端**                        |
| ---------------------------- | ------------------------------ |
| 1. 创建 Socket（ `socket()` ）   | 1. 创建 Socket（ `socket()` ）     |
| 2. 绑定 IP 和端口（ `bind()` ）     | 2. 连接服务端（ `connect()` ）        |
| 3. 监听连接（`listen()`）          |                                |
| 4. 接受连接（`accept()`）          |                                |
| 5. 读写数据（`send()` / `recv()`） | 3. 读写数据（ `send()` / `recv()` ） |
| 6. 关闭 Socket（ `close()` ）    | 4. 关闭 Socket（ `close()` ）      |


### 关键技术点

#### 三次握手（TCP）

Socket 连接建立时，客户端与服务端交换 SYN/ACK 包确保通道可靠。

#### 缓冲区机制

Socket 发送/接收数据时依赖内核缓冲区，可能阻塞（需处理 `EAGAIN` 等错误）。

#### [多路复用](IO%20多路复用.md)

通过多路复用（Multiplexing）实现高并发是网络编程中的核心技术，其**核心思想**是**用一个进程/线程监听多个 Socket 事件**，避免为每个连接创建独立线程/进程的资源消耗。

**Linux 首选 `epoll` ，Windows 可用 `select` 或 IOCP （异步IO）。**


#### NAT穿透

公网与内网 Socket 通信需解决地址转换问题（如 STUN/TURN 协议）。


### 常见问题

- **端口占用**：`Address already in use`（可通过`SO_REUSEADDR`选项解决）。
- **粘包问题**：TCP 是流协议，需自定义边界（如长度前缀或分隔符）。
- **超时控制**：设置 `SO_TIMEOUT` 避免无限阻塞。


#### 处理粘包

在通信协议上，处理粘包的方法有 2 种：
1. 文本协议：使用换行为包结束标志
2. 二进制协议：使用长度+数据的方式定义一个包


二进制协议中，同步 IO 和 Proactor 模型处理粘包方式为：
- 先读取包头，然后读取剩余部分
```C++
int read_a_packet(int fd, char* buffer)
{
    int pkt_size = 0;
    read(fd, &pkt_size, sizeof(int));
    int read_size = 0;
    do {
        int read_ret = read(fd, buffer + read_size, pkt_size - read_size);
        read_size += read_ret;
    } while (read_size < pkt_size);
    
    return pkt_size;
}
```

- 异步 IO 只需要将 `read` 换成 `co_await async_read` ，其他的逻辑是完全一致的。








## 使用示例

### C++


#### Linux 基础库

`<sys/socket.h>`

![](imgs/Pasted%20image%2020230704134654.png)
 
 **`sockaddr`**：socket 地址结构体
```C++
struct in_addr
{
	in_addr_t       s_addr;         // 32位IPv4地址
};

struct sockaddr_in
{
	uint8_t         sin_len;        // 结构长度，非必需
	sa_family_t     sin_family;     // 地址族，一般为AF_****格式，常用的是AF_INET
	in_port_t       sin_port;       // 16位TCP或UDP端口号
	struct in_addr  sin_addr;       // 32位IPv4地址
	char            sin_zero[8];    // 保留数据段，一般置零
};
```

1. **`socket(...)`**：创建 socket
    - `family`：`AF_INET`（IPv4）、`AF_INET6`（IPv6）
    - `type`：
	    - `SOCK_STREAM` 字节流 socket，适用于 TCP 或 SCTP 协议；
	    - `SOCK_DGRAM` 数据报 socket，适用于UDP协议；
	    - `SOCK_SEQPACKET` 有序分组 socket，适用于SCTP协议；
	    - `SOCK_RAW` 原始套接字，适用于绕过传输层直接与网络层协议（IPv4/IPv6）通信。
    - `protocol`：`IPPROTO_TCP`使用 TCP 
2. **`bind(...)`**：
    - 用于将 socket 与一个 `ip::port` 绑定，或者更应该说是把一个本地协议地址赋予一个 socket。
3. **`listen(...)`** ：
    - 开启 socket 的监听状态，也就是将 socket 从 `CLOSE` 状态转换为 `LISTEN` 状态。
    - backlog：Accept 队列的最大长度，可调参数。
4. **`connect(...)`** ：
    - 用于客户端跟绑定了指定的 ip 和 port 并且处于 LISTEN 状态的服务端进行连接。
5. **`accept(...)`** ：
    - 由 TCP 服务器调用，用于跟客户端建立连接，并返回客户端 socket。
6. **`recv(...)`** ：用于通过 socket 接收数据
7. **`send(...)`** ：用于通过 socket 发送数据
8. **`close(...)`**：用于关闭 socket ，并终止 TCP 连接。

#### 简单 CS 模型
![](imgs/Pasted%20image%2020230704134708.png)


##### 服务端

```C++
// server.cpp

#include <iostream>
#include <cstring>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 16555
#define BUFFSIZE 2048
#define MAX_CLIENTS 5

int main() {
    int server_fd, new_socket;
    struct sockaddr_in address;
    int opt = 1;
    int addrlen = sizeof(address);
    char buffer[BUFFER_SIZE] = {0};

    // 1. 创建socket文件描述符
    if (0 == (server_fd = socket(AF_INET, SOCK_STREAM, 0))) {
        perror("socket failed");
        exit(EXIT_FAILURE);
    }

    // 2. 设置socket选项(避免地址使用错误)
    if (setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR | SO_REUSEPORT, &opt, sizeof(opt))) {
        perror("setsockopt");
        exit(EXIT_FAILURE);
    }

    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);

    // 3. 绑定socket到端口
    if (0 > bind(server_fd, (struct sockaddr *)&address, sizeof(address))) {
        perror("bind failed");
        exit(EXIT_FAILURE);
    }

    // 4. 监听连接
    if (0 > listen(server_fd, MAX_CLIENTS)) {
        perror("listen");
        exit(EXIT_FAILURE);
    }

    std::cout << "Server listening on port " << PORT << std::endl;

    // 5. 主服务循环
    while (true) {
        std::cout << "Waiting for new connection..." << std::endl;

        // 6. 接受客户端连接
        if (0 > (new_socket = accept(server_fd, (struct sockaddr *)&address, (socklen_t*)&addrlen))) {
            perror("accept");
            continue; // 继续等待下一个连接
        }

        // 打印客户端信息
        char client_ip[INET_ADDRSTRLEN];
        inet_ntop(AF_INET, &(address.sin_addr), client_ip, INET_ADDRSTRLEN);
        std::cout << "Connection accepted from " << client_ip << ":" << ntohs(address.sin_port) << std::endl;

		// 7. 读取客户端消息
        int valread = read(new_socket, buffer, BUFFER_SIZE - 1);
        if (0 >= valread) {
            std::cerr << "Error reading from client or connection closed" << std::endl;
            close(new_socket);
            continue;
        }

        buffer[valread] = '\0'; // 确保字符串正确终止
        std::cout << "Message from client: " << buffer << std::endl;

        // 8. 发送响应
        const char *response = "Hello from server";
        if (0 > send(new_socket, response, strlen(response), 0)) {
            std::cerr << "Failed to send response" << std::endl;
        } else {
            std::cout << "Response sent to client" << std::endl;
        }

        // 9. 关闭当前客户端连接
        close(new_socket);
        std::cout << "Connection closed" << std::endl;
    }

    // 理论上不会执行到这里
    close(server_fd);

    return 0;
}
```

##### 客户端

```C++
// client.cpp

#include <iostream>
#include <cstring>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define PORT 8080

int main() {
    int sock = 0;
    struct sockaddr_in serv_addr;
    char buffer[1024] = {0};
    
    // 1. 创建socket
    if (0 > (sock = socket(AF_INET, SOCK_STREAM, 0))) {
        std::cerr << "Socket creation error" << std::endl;
        return -1;
    }
    
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(PORT);
    
    // 2. 将IPv4地址从文本转换为二进制形式
    if (0 >= inet_pton(AF_INET, "127.0.0.1", &serv_addr.sin_addr)) {
        std::cerr << "Invalid address/ Address not supported" << std::endl;
        return -1;
    }
    
    // 3. 连接到服务器
    if (0 > connect(sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr))) {
        std::cerr << "Connection Failed" << std::endl;
        return -1;
    }
    
    // 4. 发送消息
    const char *hello = "Hello from client";
    send(sock, hello, strlen(hello), 0);
    std::cout << "Hello message sent" << std::endl;
    
    // 5. 读取服务器响应
    int valread = read(sock, buffer, 1024);
    std::cout << "Server says: " << buffer << std::endl;
    
    // 6. 关闭socket
    close(sock);
    
    return 0;
}
```

