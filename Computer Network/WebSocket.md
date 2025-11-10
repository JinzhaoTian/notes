早期，很多网站为了实现推送技术，所用的技术都是轮询（也叫短轮询），是指由浏览器每隔一段时间向服务器发出 HTTP 请求，然后服务器返回最新的数据给客户端。这种模式有很明显的缺点，即浏览器需要不断的向服务器发出请求，然而 HTTP 请求与响应可能会包含较长的头部，其中真正有效的数据可能只是很小的一部分，所以这样会消耗很多带宽资源。

WebSocket 是一种网络传输协议，可在单个 TCP 连接上进行全双工通信，位于 OSI 模型的应用层。WebSocket 协议在 2011 年由 IETF 标准化为 RFC 6455，后由 RFC 7936 补充规范。

WebSocket 使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在 WebSocket API 中，浏览器和服务器只需要完成一次握手，两者之间就可以创建持久性的连接，并进行双向数据传输。

WebSocket 使用 ws 或 wss 的统一资源标志符（URI），其中 wss 表示使用了 TLS 的 WebSocket。默认情况下 WebSocket 协议使用 80 端口，若运行在 TLS 之上时，默认使用 443 端口。

## WebSocket 协议库

- C++: [uWebSockets](https://github.com/uNetworking/uWebSockets)  
- Python: [websockets](https://pypi.org/project/websockets/)  
