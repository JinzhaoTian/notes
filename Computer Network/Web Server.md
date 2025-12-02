Web Server（Web 服务器） 一词可以代指硬件或软件，或者是它们协同工作的整体。

1. **硬件部分**：Web 服务器是一台存储了 Web 服务器软件以及网站的组成文件（比如 HTML 文档、图片、CSS 样式表和 JavaScript 文件）的计算机。它接入到互联网并且支持与其他连接到互联网的设备进行物理数据的交互。
2. **软件部分**：Web 服务器包括控制网络用户如何访问托管文件的几个部分，至少是一台 HTTP 服务器。一台 HTTP 服务器是一种能够理解 [URL](URL.md) 和 [HTTP](HTTP.md) 的软件。一个 HTTP 服务器可以通过它所存储的网站域名进行访问，并将这些托管网站的内容传递给最终用户的设备。

当浏览器需要一个托管在网络服务器上的文件的时候，浏览器通过 HTTP 请求这个文件。当这个请求到达正确的 Web 服务器（硬件）时，HTTP 服务器（软件）收到这个请求，找到这个被请求的文档（如果这个文档不存在，那么将返回一个 404 响应），并把这个文档通过 HTTP 发送给浏览器。

![](_imgs/Pasted%20image%2020251202142422.png)



在网络框架中，Web 服务器负责接收来自客户端的 HTTP 请求，并将其传递给应用程序代码处理，然后将处理结果以 HTTP 响应的形式返回给客户端。

## 主要功能

1. 监听网络端口接收 HTTP 请求
2. 路由请求到相应的处理程序
3. 管理连接和并发
4. 返回 HTTP 响应
5. 处理 SSL/TLS 加密

## 常见框架

1. **Java Spring 生态系统**
	- **[Tomcat](../Language/Java/Tomcat.md)**：最常用的 Servlet 容器
	- **Jetty**：轻量级替代品
	- **Undertow**：高性能、非阻塞服务器
	- **[Netty](../Language/Java/Netty.md)**：底层网络框架（更底层，可用于构建自定义服务器）

2. **Node.js 生态系统**
	- **内置HTTP模块**：Node.js 核心模块，提供基础 HTTP 服务
	- **[Express](../Language/Node/后端开发/Express.md)**：Web 应用框架（包含 HTTP 服务器功能）
	- **[Fastify](../Language/Node/后端开发/Fastify.md)**：高性能 HTTP 服务器框架

3. **ASP.NET Core**
	- **[Kestrel](../Language/CS/后端/Kestrel.md)**：ASP.NET Core 内置的跨平台、高性能 Web 服务器


## 现代趋势

1. **内嵌服务器模式**：现代框架（ASP.NETCore、Spring Boot）倾向内置 Web 服务器
2. **云原生设计**：轻量级、容器友好（Kestrel、Undertow）
3. **边缘计算**：将 Web 服务器功能部署到 CDN 边缘节点
4. **无服务器**：AWS Lambda、Azure Functions 等抽象了服务器管理