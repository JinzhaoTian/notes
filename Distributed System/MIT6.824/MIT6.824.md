MIT 6.824（现更名为6.5840）是麻省理工学院（MIT）开设的一门研究生级别的分布式系统课程，由 PDOS 实验室（Parallel and Distributed Operating Systems group）提供。

该课程以理论与实践相结合的方式，深入探讨分布式系统的设计原理、容错机制、一致性协议以及高性能计算架构。

## 课程内容与结构

1. **核心主题**：课程涵盖分布式系统的关键概念，如
	- 容错（fault tolerance）
	- 复制（replication）
	- 一致性（[Consistency](../Consistency.md)）
	- 并发控制（[Concurrency Control](../Concurrency%20Control.md)）
	- 分布式事务（distributed transactions）

2. **教学方式**：采用“论文阅读+视频讲解+实验编程”的模式，每节课围绕一篇经典分布式系统论文展开，如 Google 的 [MapReduce](../../Backend/分布式中间件/MapReduce.md) 、[Raft](../Raft.md) 共识算法等。

3. **实验（Labs）**：课程包含 4 个渐进式实验项目，使用 Go 语言实现：
    - **MapReduce**：实现分布式计算框架，支持并行处理大数据任务
    - **Raft**：实现分布式共识算法，支持领导选举、日志复制和故障恢复
    - **容错键值存储（Fault-tolerant KV Service）**：基于 Raft 构建强一致性的分布式数据库
    - **分片键值存储（Sharded KV Service）**：扩展至分片架构，优化负载均衡

## 学习资源

- [**课程网站**](https://pdos.csail.mit.edu/6.824/schedule.html) 
- [**视频资源**](https://www.bilibili.com/video/BV1R7411t71W) 
- **社区支持**：GitHub 上有大量开源实验实现（如[PKUFlyingPig/MIT6.824](https://github.com/PKUFlyingPig/MIT6.824)），但课程建议独立完成以避免依赖。