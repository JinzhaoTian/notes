[brpc](https://github.com/apache/brpc/blob/master/README_cn.md) （**better RPC**）是百度开源的一款高性能、工业级 RPC 框架，专为微服务和分布式系统设计，常用于搜索、存储、机器学习、广告、推荐等高性能系统。最初用于百度内部大规模分布式系统，后开源并成为 Apache 孵化项目（Apache bRPC）。


## 核心特性

1. **高性能**
	- 基于 [C++](../../../Language/C++/C++.md) 开发，性能接近原生代码，适用于高并发场景。
	- 支持多种协议，兼容性强。
		- HTTP：兼容 [RESTful API](RESTful%20API.md) ，支持浏览器访问
		- [gRPC](gRPC.md)：支持 Google gRPC 协议
		- Thrift：兼容 Apache Thrift
		- [Redis](../../../Database/Redis/Redis.md)：直接与 Redis 服务交互。
		- 自定义协议：灵活扩展私有协议
	- 单机可支持百万级 [QPS](../系统指标/QPS.md)（百度内部验证）。

2. **丰富的扩展能力**
	- **负载均衡**：支持轮询、随机、[一致性哈希](../../../Distributed%20System/Consistent%20Hashing.md)等策略。
	- **服务发现**：集成 [ZooKeeper](../分布式中间件/ZooKeeper.md) 、[Nacos](../架构设计/Nacos.md) 、[etcd](../分布式中间件/etcd.md) 等。
	- **熔断降级**：自动隔离故障节点，防止雪崩。
	- **流量控制**：[限制请求速率](../架构设计/限流.md)保护服务稳定性。