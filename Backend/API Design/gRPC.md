Google 于 2015 年发布了开源 RPC 框架 gRPC（**gRPC Remote Procedure Calls**），它是一个高性能、开源的的通用 [RPC](../../../Computer%20Network/RPC.md) 框架，专为微服务、分布式系统设计，支持跨语言通信（如 Go、Java、Python、C++ 等），并提供了强大的功能如双向流、认证和负载均衡。


![](imgs/Pasted%20image%2020240321221830.png)

随着微服务架构和云原生架构的出现，传统的单体应用程序被分解为一组细粒度的、自治的和面向业务能力的”微服务”，网络通信链路的数量激增，进程间（或服务间/应用程序间）通信技术也因此成为了现代分布式系统中至关重要的一个环节。

目前，最常见最传统的进程间通信方式是构建一个 Restful 服务，将应用程序建模为一个可访问的资源集合，然后通过 http 协议进行服务调用，获取资源或者变更资源状态。然而，在比较多的场景下Restful 服务对于构建进程间通信来说过于庞大、低效且容易出错，需要一个比 Restful 服务更高效的高可扩展、松耦合的进程间通信技术。因此，诞生了 gRPC ，**一种用于构建分布式应用程序和微服务的现代进程间通信方式**。

![](imgs/Pasted%20image%2020240321221607.png)




使用 gRPC，我们只需要定义好每个 API 的Request 和 Response，剩下的 gRPC 这个框架会帮我们自动搞定。

gRPC 的典型特征就是使用 [ProtoBuf](../../../Language/Data%20Format/ProtoBuf.md) 作为其接口定义语言（Interface Definition Language，IDL），同时底层的消息交换格式也是使用 protobuf。

## 核心特性

1. **基于 HTTP/2**
	- 支持 [**IO 多路复用**](../../../Computer%20Network/IO%20多路复用.md) （多个请求/响应在同一连接上并行传输）。
	- **二进制协议**，比 HTTP/1.1 更高效（头部压缩、减少延迟）。
	- 支持**服务器推送**和**双向流式通信**。

2. **使用 Protocol Buffers（[ProtoBuf](../../../Language/Data%20Format/ProtoBuf.md)）**
	- **高效序列化**：二进制格式，比 JSON/XML 更小、更快。
	- **强类型接口定义**：通过 `.proto` 文件定义服务和方法，自动生成代码。
	- **跨语言支持**：只需定义一次接口，即可生成多种语言的客户端/服务端代码。

3. **四种通信模式**
	- **Unary（一元）**：类似传统 RPC：客户端发送一个请求，服务端返回一个响应（如 `add(a, b)`）。
	- **Server Streaming** ：客户端发送一个请求，服务端返回多个响应（如实时数据推送）。
	- **Client Streaming** ：客户端发送多个请求，服务端返回一个响应（如文件上传）。
	- **Bidirectional Streaming** ：客户端和服务端均可异步发送多个请求/响应（如聊天应用）。

4. **跨平台 & 多语言支持**
	- 官方支持10+ 语言（Go、Java、Python、C#、Node.js 等）。
	- 适用于**微服务**、**云原生应用**（如 [Kubernetes](../../DevOps/Container/Kubernetes.md) 内部通信）。

5. **内置高级功能**
	- **认证**（SSL/TLS、Token-based）。
	- **负载均衡**（客户端/服务端）。
	- **超时重试**、**拦截器**（Middleware）。



## 通信流程

1. **定义接口**（`.proto` 文件）：
```protobuf
service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}
message HelloRequest { string name = 1; }
message HelloReply { string message = 1; }
```

2. **生成代码**：编译 `.proto` 文件，得到存根（stub）文件（即客户端和服务端代码）

3. **实现服务端**：服务端（gRPC Server）实现定义的接口，编写业务逻辑，并启动，处理客户端请求

4. **客户端调用**：直接调用 `SayHello`，如同本地方法


以上就是 gRPC 的基本流程，由于 `.proto` 文件的编译支持多种语言（Go、Java、Python等），所以 gRPC 也是跨语言的。


## 适用场景

1. **微服务通信**：服务间高效调用（如 Kubernetes Pods 间通信）。
2. **实时应用**：聊天、游戏、股票行情（利用双向流）。
3. **跨语言系统**：如 Go 服务调用 Python 算法模块。
4. **低延迟场景**：物联网（IoT）、金融交易。



## 使用示例


### Python

1. **定义服务**（ `greeter.proto` ）
```protobuf
syntax = "proto3";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

2. **生成代码**
```bash
python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. greeter.proto
```

3. **服务端实现**
```python
import grpc
from concurrent import futures
import greeter_pb2, greeter_pb2_grpc

class Greeter(greeter_pb2_grpc.GreeterServicer):
    def SayHello(self, request, context):
        return greeter_pb2.HelloReply(message=f"Hello, {request.name}!")

server = grpc.server(futures.ThreadPoolExecutor())
greeter_pb2_grpc.add_GreeterServicer_to_server(Greeter(), server)
server.add_insecure_port("[::]:50051")
server.start()
server.wait_for_termination()
```

4. **客户端调用**
```python
import grpc
import greeter_pb2, greeter_pb2_grpc

channel = grpc.insecure_channel("localhost:50051")
stub = greeter_pb2_grpc.GreeterStub(channel)
response = stub.SayHello(greeter_pb2.HelloRequest(name="World"))
print(response.message)  # 输出: "Hello, World!"
```