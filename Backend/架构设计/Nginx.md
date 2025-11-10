Nginx 在微服务架构中扮演着多功能的角色，从请求路由、负载均衡、反向代理，到缓存、认证、限流和日志记录，它为系统提供了高性能、可扩展和安全的解决方案。通过适当配置 Nginx，可以显著简化微服务的管理，提高整个系统的可靠性和用户体验。

## 使用场景

1. **API 网关**
	- **作用**：Nginx 作为 API 网关，充当客户端与后端微服务之间的中间层。它负责请求的路由、负载均衡、协议转换、认证和授权等功能。
	- **如何使用**：通过配置 Nginx，将客户端请求路由到适当的后端服务。可以根据 URL 路径、请求参数、HTTP 头信息等条件来决定将请求转发到哪个服务。
	- **优点**：集中管理入口请求，简化后端服务，提供一致的访问接口，提高安全性。

2. [**负载均衡**](负载均衡.md)
	- **作用**：Nginx 可以作为负载均衡器，在多个后端微服务实例之间分配流量，确保系统的高可用性和响应速度。
	- **如何使用**：配置 Nginx 以使用不同的负载均衡算法（如轮询、最少连接、IP 哈希等），将请求分配到多个后端服务器。
	- **优点**：优化资源利用，避免单点故障，提高系统的伸缩性。

3. **反向代理**
	- **作用**：作为反向代理，Nginx 接收客户端请求并将其转发给后端服务，同时将后端的响应返回给客户端。
	- **如何使用**：通过配置反向代理设置，Nginx 可以在前端和后端之间充当中介，隐藏后端服务的具体细节，保护后端服务器的安全。
	- **优点**：提高系统的安全性，隐藏内部架构，减少暴露在外部的攻击面。

4. **静态内容分发**
	- **作用**：Nginx 可以高效地分发静态内容，如 HTML、CSS、JavaScript、图像和视频文件。
	- **如何使用**：配置 Nginx 作为静态文件服务器，将静态内容直接从 Nginx 缓存或文件系统中提供，而不需要请求到达后端服务。
	- **优点**：提高响应速度，减轻后端服务的负担，优化用户体验。


## 使用示例

### 配置文件解析

#### location

每个 server 中可以包含多个 location ，在整个 Nginx 配置文档中起着重要的作用，而且 Nginx 服务器在许多功能上的灵活性往往在location指令的配置中体现出来。

location 块的主要作用是，基于 Nginx 服务器接收到的请求字符串（例如， server_name/uri-string），对除虚拟主机名称（也可以是IP别名，后文有详细阐述）之外的字符串（前例中“/uri-string”部分）进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能都是在这部分实现。许多第三方模块的配置也是在location块中提供功能。

在 Nginx 的官方文档中定义的 location 的语法结构为：

```
location [ = | ~ | ~* | ^~ ] uri { ... }
```

其中，uri 变量是待匹配的请求字符串，可以是不含正则表达的字符串，如 /myserver.php 等；也可以是包含有正则表达的字符串，如 .php$（表示以 .php 结尾的 URL）等。为了下文叙述方便，我们约定，不含正则表达的uri称为 "标准uri"，使用正则表达式的 uri 称为 "正则uri" 。

其中方括号里的部分，是可选项，用来改变请求字符串与 uri 的匹配方式。

在不添加此选项时，Nginx 服务器首先在 server 块的多个 location 块中搜索是否有标准 uri 和请求字符串匹配，如果有多个可以匹配，就记录匹配度最高的一个。然后，服务器再用 location 块中的正则 uri 和请求字符串匹配，当第一个正则 uri 匹配成功，结束搜索，并使用这个 location 块处理此请求；如果正则匹配全部失败，就使用刚才记录的匹配度最高的 location 块处理此请求。

选项中各个标识的含义：
- `=` ：用于标准 uri 前，要求请求字符串与 uri 严格匹配。如果已经匹配成功，就停止继续向下搜索并立即处理此请求。
- `^～` ：用于标准 uri 前，要求 Nginx 服务器找到标识 uri 和请求字符串匹配度最高的 location 后，立即使用此 location 处理请求，而不再使用 location 块中的正则 uri 和请求字符串做匹配。
- `～` ：用于表示 uri 包含正则表达式，并且区分大小写。
- `～*` ：用于表示 uri 包含正则表达式，并且不区分大小写。注意如果 uri 包含正则表达式，就必须要使用 `～` 或者 `～*` 标识。


### 反向代理

使用 `127.0.0.1` 或者 `localhost` 作为 `Nginx` 的反向代理地址时，Nginx 将请求转发到它运行的本地服务器。如果你的后端服务没有运行在 **Nginx 服务器本身**，使用 `127.0.0.1` 或 `localhost` 会导致 Nginx 无法正确访问后端服务。这是因为 `127.0.0.1` 指的是 **Nginx 服务器的本地网络接口**，而不是后端服务所在的服务器。

如果 `Nginx` 和后端服务位于 **不同的机器**，使用 `127.0.0.1` 会导致这些问题：

1. **Nginx 转发到了错误的机器**：
    - `127.0.0.1` 指向的是 Nginx 服务器本身，而不是运行后端服务的服务器。因此，Nginx 会尝试在自身机器上寻找端口 `3030`，而不是后端服务所在的机器。
2. **服务监听范围受限**：
    - 后端服务可能只监听在特定 IP 地址（例如 `127.0.0.1`），导致它无法接受来自其他机器的请求。如果你的后端服务配置成只监听 `127.0.0.1`，它将无法接受来自 Nginx 服务器的请求。你需要确保后端服务监听的是**所有网络接口**（即 `0.0.0.0`），这样它才能接受来自其他 IP 地址的请求。

当 `Nginx` 和后端服务都运行在同一个服务器的 Docker 容器中时，使用 `localhost` 或 `127.0.0.1` 可能不会正常工作，除非这两个容器共享网络栈。通常情况下，Docker 容器有各自独立的网络命名空间，因此一个容器中的 `localhost` 只指向该容器自身，而不是宿主机或其他容器。

也就是说，如果 Nginx 是用 Docker 起来的，内部反向代理就不能配置为  `127.0.0.1` 或者 `localhost` ，要使用局域网的 IP，走外部 Socket。

**解决方案**：

1. **使用 Docker 网络**：

创建一个自定义 Docker 网络，将 Nginx 和后端服务容器连接到同一个网络中。这样，你可以通过容器的名称进行通信，而不是使用 `localhost`。

你可以通过以下命令创建一个自定义网络：
```
docker network create my_network
```

然后在启动容器时，将它们连接到这个网络：
```
docker run --name nginx-container --network my_network -p 10083:80 -d nginx
docker run --name backend-container --network my_network -p 3030:3030 -d your-backend-image
````

在 `Nginx` 配置中，你可以使用后端服务的容器名称 `backend-container` 来代替 `localhost`：
```
server {
    listen 10083;
    server_name your_domain.com;

    location / {
        proxy_pass http://backend-container:3030;  # 使用容器名称
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

2. **使用 `host` 网络模式**（不推荐）： 如果你确实想让 `localhost` 指向宿主机，你可以将容器运行在 `host` 网络模式下：
```
docker run --network host -d your-backend-image
```
这样，容器将共享宿主机的网络栈，`localhost` 将指向宿主机。但这意味着容器的端口直接暴露在宿主机上，可能不太安全，也不适用于多容器环境。

### 配置

`docker-compose.yml` ：
```YAML
version: '3.1'
services:
  nginx:
    image: nginx:latest
    container_name: nginx
    restart: unless-stopped
    # network_mode: host
    labels: 
      - 'docker_nginx'
    environment:
      TZ: Asia/Shanghai
    volumes:
      - ./html:/usr/share/nginx/html
      - ./conf/nginx.conf:/etc/nginx/nginx.conf
      - ./conf.d:/etc/nginx/conf.d
      - ./ssl:/etc/nginx/ssl
      - ./logs:/var/log/nginx
    ports:
      - 80:80
      - 443:443
```

电脑开启80和443端口。

网关配置，`nginx.conf` 如下：

```conf
user nginx;
worker_processes auto; 

error_log   /var/log/nginx/error.log notice;
pid         /var/run/nginx.pid;

# include /usr/share/nginx/modules/*.conf;

events {
    worker_connections  1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    # access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    include /etc/nginx/conf.d/*.conf;
}

```

`default.conf` 如下：

```conf
server {
    listen      80;
    server_name example.com;

    return 301  https://$host$request_uri;

    access_log  off;
}

server {
    listen       443 ssl http2;
    listen  [::]:443 ssl http2;
    server_name  example.com;

    ssl_certificate     /etc/nginx/ssl/example.com/fullchain.cer;
    ssl_certificate_key /etc/nginx/ssl/example.com/key.pem;
    ssl_protocols       TLSv1.2 TLSv1.3;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always; 

    client_max_body_size 128M;

    location / {
        root   /usr/share/nginx/html/navi;
        index  index.html index.htm;
    }

    access_log  /var/log/nginx/host_access.log  main;
}
```

其中开启每个服务的网关设置 `example.conf` 如下：

```conf
# web

server {
    listen       80;
    server_name  web.example.com;

    return 301 https://$host$request_uri;

    access_log off;
}

server {
    listen       443 ssl http2;
    listen  [::]:443 ssl http2;
    server_name  web.example.com;

    ssl_certificate     /etc/nginx/ssl/example.com/fullchain.cer;
    ssl_certificate_key /etc/nginx/ssl/example.com/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always; 

    client_max_body_size 128M;
    
    location / {
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass http://[Inner-IP]:[Port];
    }

    access_log  /var/log/nginx/ghost_access.log  main;
}
```


**注意**：

实际部署时的 `[Inner-IP]` 要部署成服务器局域网 IP ，即使是在同一个机器，这个应该是和网络配置有关系。


## 基础架构

![](imgs/Pasted%20image%2020250702113112.png)



### 多进程 + 事件驱动

Nginx 使用 Master-Worker **多进程模型**，主进程负责管理，子进程负责处理请求：
- **Master 进程职责**：
	- **管理 Worker 进程**： Master 进程负责启动、停止和重启 Worker 进程，确保系统的稳定运行。
	- **加载配置文件**： 它负责读取和解析 Nginx 的配置文件（`nginx.conf`），并将配置信息传递给 Worker 进程。
- **Worker 进程职责**：
	- **处理客户端请求**： Worker 进程是实际处理客户端请求的进程。
	- **执行网络 I/O 操作**： 它负责接收和发送网络数据，处理 HTTP 请求和响应。
	- **每个 Worker 进程都是独立的**： 这意味着它们之间互不干扰，即使某个 Worker 进程出现问题，也不会影响其他进程的运行。


### 异步非阻塞处理机制

Nginx 通过 epoll 和非阻塞 I/O 的结合使用，Nginx 能够高效地处理大量的并发连接，使它能够在一个线程中同时处理多个连接，从而提高了系统的并发处理能力。

1. **单线程**：每个 Worker 进程都是单线程的，这避免了多线程上下文切换的开销，提高了性能。
2. [**epoll**](../../Computer%20Network/IO%20多路复用.md#epoll) ：相比于 select、poll，在大数量连接的情况下，性能会更好。
3. [**非阻塞 I/O**](../../Computer%20Network/非阻塞%20IO.md)：这意味着当线程执行 I/O 操作时，不会等待操作完成，而是立即返回。线程可以继续执行其他任务，当 I/O 操作完成后，通过事件通知机制进行处理。非阻塞 I/O 可以避免线程在等待 I/O 的时候，被阻塞。

