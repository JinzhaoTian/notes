Paxos 是由 Leslie Lamport 在 1990 年提出的分布式一致性算法，被认为是这类算法的理论基础。

## 核心特点

1. **复杂性**：Paxos 以难以理解和实现著称
2. **角色划分**：
	- 提议者（Proposer）
	- 接受者（Acceptor）
	- 学习者（Learner）
3. **两阶段过程**：
    - 准备阶段（Prepare）：提议者获取承诺
    - 接受阶段（Accept）：提议者提交值
4. **灵活性**：可以处理多个提议同时进行的情况


## 核心概念


#### 角色划分

- **提议者（Proposer）**：提出提案（proposal）
- **接受者（Acceptor）**：对提案进行投票表决
- **学习者（Learner）**：学习被选定的值

在实际实现中，一个节点可以同时扮演多个角色。

#### 提案与值

- **提案（proposal）**：包含提案编号（n）和提案值（v），记作（n, v）
- **提案编号（n）**：全局唯一且全序的编号，用于区分不同提案
- **提案值（v）**：需要达成一致的值




## 算法原理

Paxos 算法解决的是分布式一致性问题，即**在一个可能发生消息丢失、重复或乱序的异步系统中，如何就某个值（提案）达成一致**。

Lamport 提出的 Paxos 算法包括两个部分：

- Basic Paxos 算法：多节点如何就某个值达成共识
- Multi Paxos 思想：执行多个 Basic Paxos ，就一系列的值达成共识


### Basic Paxos

在 Basic Paxos 中，集群中各个节点为了达成共识，需要进行两阶段的协商，即准备阶段（Prepare Phase）和接受阶段（Accept Phase）。

![](_imgs/Pasted%20image%2020250626135329.png)
#### Prepare Phase

1. Proposer 选择一个提案编号 n，向所有或多数 Acceptor 发送 Prepare（n）请求
2. Acceptor 收到 Prepare（n）请求后：
    - 如果 n 大于它已响应的所有 Prepare 请求的编号
        - 承诺不再接受任何编号小于 n 的提案
        - 返回它已接受的编号最大的提案（如果有）
    - 否则拒绝请求

#### Accept Phase

1. 如果 Proposer 收到了多数 Acceptor 对 Prepare（n）的响应：
    - 如果所有响应都没有返回已接受的提案，Proposer 使用自己的值 v
    - 如果有返回已接受的提案，Proposer 使用其中编号最大的提案的值
    - Proposer 向所有或多数 Acceptor 发送 Accept（n，v）请求
2. Acceptor 收到 Accept（n，v）请求后：
    - 如果没有承诺过不接受编号为 n 的提案（即没有响应过更高编号的 Prepare 请求）
        - 接受该提案
        - 返回接受响应
    - 否则拒绝请求
3. 如果 Proposer 收到多数 Acceptor 的接受响应，则值 v 被选定（chosen）
4. Learner 发现值被选定后，学习该值


### Multi Paxos

Basic Paxos 算法只适用于对单个值情形，不适用于多个值情形，因此 Basic Paxos 算法几乎只是用来理论研究，并不直接应用在实际工作中。

为此，Lamport 提出 Multi Paxos 思想，基于 Multi Paxos 思想，通过多个 Basic Paxos 实例实现一系列值的共识的算法统称为 Multi Paxos 算法。

如果直接通过多次执行 Basic Paxos 实例方式，来实现一系列值的共识，存在以下问题：
- 如果集群中多个提议者同时在准备阶段提交提案，可能会出现没有提议者接收到大多数准备响应，导致需要重新提交准备请求。例如，在一个 5 个节点的集群中，有 3 个节点同时作为提议者同时提交提案，那就会出现没有一个提议者获取大多数的准备响应，而需要重新提交
- 为了达成一个值的共识，需要进行 2 轮 RPC 通讯，分别是准备阶段和接受阶段，性能低下

为了解决以上问题，Multi Paxos 引入了领导者（Leader）和优化了 Basic Paxos 的执行过程。

![](_imgs/Pasted%20image%2020250626135242.png)



#### Leader Election

存在多个提议者同时提交准备请求的情况，如果引入了领导者，由领导者作为唯一的提议者，就可以解决冲突问题。


#### Accept Phase

引入领导者后，只有领导者才可发送提议，因此，领导者的提案就已经是最新的了，不再需要通过准备阶段来发现之前被大多数节点通过的提案，领导者可以独立指定提议的值。

这样一来，准备阶段存在就没有意义了，领导者可以直接跳过准备阶段，直接进行接受阶段，减少了 RPC 通讯次数。




## 算法实现

### [Raft](Raft.md)



### ZAB

在 ZooKeeper 中，Paxos 具体的实现是 [ZAB](../Backend/分布式中间件/ZooKeeper.md#ZAB) ，是专为 ZooKeeper 设计的一种支持崩溃恢复的原子广播协议，Zookeeper 主要依赖 ZAB 协议来实现分布式数据一致性。

### Chubby

Google 分布式锁 Chubby 实现了 Multi Paxos 算法，主要包括：
- Chubby 引入主节点作为领导者，即主节点作为唯一提议者，不存在多个提议者同时提交提案的情况，也不存在提案冲突的情况。Chubby 通过执行 Basic Paxos 算法进行投票选举产生主节点
- 在 Chubby 中，由于引入了主节点，因此，也去除了 Basic Paxos 的准备阶段
- 在 Chubby 中，为实现强一致性，所有的读请求和写请求都由主节点来处理


> Google Chubby 的作者 Mike Burrows 说过这个世界上只有一种一致性算法，那就是 Paxos，其它的算法都是残次品。


