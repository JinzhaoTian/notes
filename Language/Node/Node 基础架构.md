
### 基础架构

Node.js 是一个服务器端 JavaScript 执行环境，提供了底层服务器功能环境，包括二进制数据操作、文件系统 I/O、数据库访问、网络访问等。Node.js 使用了一个事件驱动、非阻塞式 I/O 的模型，使其轻量又高效。

![](Pasted%20image%2020230704134259.png)

Node.js 主要由 [V8 引擎](Basic/V8%20引擎.md)、[libuv](../../Computer%20Network/IO库/libuv.md) 和第三方库组成：
- libuv：提供异步功能的 C 库，在运行时负责一个事件循环（Event Loop）、一个线程池（Thread Pool）、文件系统 I/O、DNS 相关和网络 I/O，以及一些其他重要功能。
- 第三方库：异步 DNS 解析（ `cares` ）、HTTP 解析器（旧版使用 `http_parser`，新版使用 `llhttp` ）、HTTP2 解析器（ `nghttp2` ）、 解压压缩库( `zlib` )、加密解密库( `openssl` )等等。


Node.js 不再为每个请求开启一个新的线程，而是**所有请求都在单一的主线程中处理**，也只做这么一件事情：处理请求（请求中包含的 I/O 操作如文件系统访问、数据库读写等，都会转发给由 libuv 管理的工作线程去执行）。也就是说，请求中的 I/O 操作是异步处理的，而非在主线程上进行。这个办法就使得主线程从不会阻塞，因为所有耗时的任务都分配到了别处。你需要面对的只有唯一的主线程，所有 libuv 管理的工作线程都与你隔离开来，无需操心，Node.js 会处理好那部分。


#### [事件循环](../../Operation%20System/异步编程.md#事件循环)

##### 浏览器事件循环

![](_imgs/Pasted%20image%2020250623164658.png)

##### Node 事件循环

![](_imgs/Pasted%20image%2020250623164636.png)
##### 微任务和宏任务

- **宏任务**（ macrotask，也叫 **tasks** ）， 一些异步任务的回调会依次进入**宏任务队列**（ macro task queue），等待后续被调用，这些异步任务包括：
	- `setTimeout`
	- `setInterval`
	- `I/O`
	- `setImmediate`（Node独有）
	- `requestAnimationFrame`（浏览器独有）
	- `UI rendering`（浏览器独有）
 

- **微队列**（microtask，也叫 **jobs** ），另外一些异步任务的回调会依次进入**微任务队列**（micro task queue），等待后续被调用，这些异步任务包括：
	- `process.nextTick`（Node独有）
	- `Promise`
	- `Object.observe`
	- `MutationObserver`





#### 启动流程

![](Pasted%20image%2020230704134309.png)

一个 Node.js 应用启动时，V8 引擎会执行你写的应用代码，保持一份观察者（注册在事件上的处理函数）列表。当事件发生时，它的处理函数会被加进一个**事件队列**（**Event Queue**）。只要这个队列还有等待执行的事件，**事件循环**（**Event Loop**）就会持续把事件从队列中拿出，放进**调用堆栈**。需要注意的是，只有当前一个事件处理完毕（调用堆栈也已经清空），事件循环才会把下一个事件放进调用堆栈。

在调用堆栈中，所有的 I/O 请求都会转发给 libuv 处理。libuv 会维持一个线程池，包含四个工作线程。文件系统 I/O 请求和 DNS 相关请求都会放进这个线程池处理；其他的请求，如网络、平台特性相关的请求会分发给相应的系统处理单元。

安排给线程池的这些 I/O 操作由 Node.js 的底层库执行，完成之后 libuv 把此事件放回事件队列，等待主线程执行后续操作。在 libuv 处理这些异步 I/O 操作期间，主线程不会等待处理结果，而是继续忙其他事情，只有当事件循环把 libuv 返回的事件放进调用堆栈之后，主线程才会继续处理这个事件的后续操作。 这就是一个事件在 Node.js 中执行的整个生命周期。


#### 进程机制与线程机制

Node.js 中的进程是使用 fork+exec 模式创建的，fork 就是复制主进程的数据，exec 是加载新的程序执行。

Node.js 提供了异步和同步创建进程两种模式。
1. 异步方式：异步方式就是创建一个子进程后，主进程和子进程独立执行，互不干扰。主进程会记录子进程的信息，子进程退出的时候会用到。
2. 同步方式：同步创建子进程会导致主进程阻塞。


Node.js 是单线程的，为了方便用户处理耗时的操作，Node.js 在支持多进程之后，又支持了多线程。Node.js 中多线程的架构如下图所示，每个子线程本质上是一个独立的事件循环，但是所有的线程会共享底层的 Libuv 线程池。

  ![](Pasted%20image%2020230704134332.png)
![](Pasted%20image%2020230704134337.png)
  

#### 进程通信与线程通信

一般进程间通信的方式有很多种，管道、信号、共享内存等等。但是 Node.js 选取的进程间通信方式是 Unix 域，因为只有 Unix 域支持文件描述符传递，文件描述符传递是一个非常重要的能力。

![](Pasted%20image%2020230704134407.png)

具体实现：

1. Node.js 底层通过 socketpair 创建两个文件描述符，主进程拿到其中一个文件描述符，并且封装 send和 on meesage 方法进行进程间通信。
2. 接着主进程通过环境变量把另一个文件描述符传给子进程。
3. 子进程同样基于文件描述符封装发送和接收数据的接口。这样两个进程就可以进行通信了。

![](Pasted%20image%2020230704134416.png)


进程的地址空间是独立的，不能直接通信，但是线程的地址是共享的，所以可以基于进程的内存直接进行通信。

![](Pasted%20image%2020230704134430.png)
![](Pasted%20image%2020230704134438.png)


  

#### 核心库
![](Pasted%20image%2020230704134454.png)

Node.js 源码仓库的 `/deps` 目录下有十几个依赖，其中既有 C 语言编写的模块（如 `libuv`、`v8`）也有 JavaScript 语言编写的模块（如 acorn、acorn-plugins）。一般重要的若干库如下：

- **`v8`**：C 语言编写，JavaScript 引擎。
- **`uv`**：也叫 `libuv` ，C 语言编写，采用非阻塞型的 I/O 操作，为 Node.js 提供了访问系统资源的能力。
- **`acorn`**：用 JavaScript 编写的轻量级 JavaScript 解析器。
- **`llhttp`**：C 语言编写，轻量级的 http 解析器。
- **`nghttp2`**/**`nghttp3`**/**`ngtcp2`**：处理 HTTP/2、HTTP/3、TCP/2 协议。
- **`openssl`**：C 语言编写，加密相关的模块，在 tls 和 crypto 模块中都有使用。
- **`node-inspect`**：让 Node.js 程序支持 CLI debug 调试模式。
- **`npm`**：JavaScript 编写的 Node.js 模块管理器。

  

### Node.js core API

[Node.js Documentation](https://nodejs.org/dist/latest-v16.x/docs/api/documentation.html)

一般分为如下几种：Buffer，Cluster，DNS，Events，File System，HTTP/HTTP2/HTTPS，Net，OS，PATH，Process，Query string，Stream，Worker threads，Zlib等。

一般分为相应的使用方法。


