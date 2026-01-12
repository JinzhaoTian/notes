
## 基础架构

Node.js 是一个服务器端 JavaScript 执行环境，提供了底层服务器功能环境，包括二进制数据操作、文件系统 I/O、数据库访问、网络访问等。Node.js 使用了一个事件驱动、非阻塞式 I/O 的模型，使其轻量又高效。

![](_imgs/Pasted%20image%2020230704134259.png)

Node.js 主要由 [V8 引擎](运行原理/V8%20引擎.md)、[libuv](../../Computer%20Network/IO库/libuv.md) 和第三方库组成：
- libuv：提供异步功能的 C 库，在运行时负责一个事件循环（Event Loop）、一个线程池（Thread Pool）、文件系统 I/O、DNS 相关和网络 I/O，以及一些其他重要功能。
- 第三方库：异步 DNS 解析（ `cares` ）、HTTP 解析器（旧版使用 `http_parser`，新版使用 `llhttp` ）、HTTP2 解析器（ `nghttp2` ）、 解压压缩库( `zlib` )、加密解密库( `openssl` )等等。


Node.js 不再为每个请求开启一个新的线程，而是**所有请求都在单一的主线程中处理**，也只做这么一件事情：处理请求（请求中包含的 I/O 操作如文件系统访问、数据库读写等，都会转发给由 libuv 管理的工作线程去执行）。也就是说，请求中的 I/O 操作是异步处理的，而非在主线程上进行。这个办法就使得主线程从不会阻塞，因为所有耗时的任务都分配到了别处。你需要面对的只有唯一的主线程，所有 libuv 管理的工作线程都与你隔离开来，无需操心，Node.js 会处理好那部分。




## 启动流程

![](_imgs/Pasted%20image%2020230704134309.png)

一个 Node.js 应用启动时，V8 引擎会执行你写的应用代码，保持一份观察者（注册在事件上的处理函数）列表。当事件发生时，它的处理函数会被加进一个**事件队列**（**Event Queue**）。只要这个队列还有等待执行的事件，**事件循环**（**Event Loop**）就会持续把事件从队列中拿出，放进**调用堆栈**。需要注意的是，只有当前一个事件处理完毕（调用堆栈也已经清空），事件循环才会把下一个事件放进调用堆栈。

在调用堆栈中，所有的 I/O 请求都会转发给 libuv 处理。libuv 会维持一个线程池，包含四个工作线程。文件系统 I/O 请求和 DNS 相关请求都会放进这个线程池处理；其他的请求，如网络、平台特性相关的请求会分发给相应的系统处理单元。

安排给线程池的这些 I/O 操作由 Node.js 的底层库执行，完成之后 libuv 把此事件放回事件队列，等待主线程执行后续操作。在 libuv 处理这些异步 I/O 操作期间，主线程不会等待处理结果，而是继续忙其他事情，只有当事件循环把 libuv 返回的事件放进调用堆栈之后，主线程才会继续处理这个事件的后续操作。 这就是一个事件在 Node.js 中执行的整个生命周期。



## 核心库
![](_imgs/Pasted%20image%2020230704134454.png)

Node.js 源码仓库的 `/deps` 目录下有十几个依赖，其中既有 C 语言编写的模块（如 `libuv`、`v8`）也有 JavaScript 语言编写的模块（如 acorn、acorn-plugins）。一般重要的若干库如下：

- **`v8`**：C 语言编写，JavaScript 引擎。
- **`uv`**：也叫 `libuv` ，C 语言编写，采用非阻塞型的 I/O 操作，为 Node.js 提供了访问系统资源的能力。
- **`acorn`**：用 JavaScript 编写的轻量级 JavaScript 解析器。
- **`llhttp`**：C 语言编写，轻量级的 http 解析器。
- **`nghttp2`**/**`nghttp3`**/**`ngtcp2`**：处理 HTTP/2、HTTP/3、TCP/2 协议。
- **`openssl`**：C 语言编写，加密相关的模块，在 tls 和 crypto 模块中都有使用。
- **`node-inspect`**：让 Node.js 程序支持 CLI debug 调试模式。
- **`npm`**：JavaScript 编写的 Node.js 模块管理器。

  

## Node.js core API

[Node.js Documentation](https://nodejs.org/dist/latest-v16.x/docs/api/documentation.html)

一般分为如下几种：Buffer，Cluster，DNS，Events，File System，HTTP/HTTP2/HTTPS，Net，OS，PATH，Process，Query string，Stream，Worker threads，Zlib等。

一般分为相应的使用方法。


