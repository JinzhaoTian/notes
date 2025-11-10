Kafka 是一种高吞吐量、持久性、分布式的**发布订阅**的[消息队列](../消息队列/消息队列.md)系统，基于 [Java NIO](../../Language/Java/Java%20NIO.md) ，主要用于处理消费者规模网站中的所有**动作流数据**，动作指网页浏览、搜索和其它用户行动所产生的数据。

常见的消息系统中所使用的消息模式如下两种：
1. Peer-to-Peer（Queue）：消息队列，生产者消费者模式。
2. Publish/Subscribe（Topic）：发布/订阅模式，发布者 Publisher 将消息发布到主题 Topic 中，同时有多个消息消费者 Subscriber 消费该消息。

Kafka 所采用的就是发布/订阅模式，被称为一种高吞吐量、持久性、分布式的发布订阅的消息队列系统。

### 三大特点

1. **高吞吐量**：可以满足每秒百万级别消息的生产和消费。
2. **持久性**：有一套完善的消息存储机制，确保数据高效安全且持久化。
3. **分布式**：Kafka 的数据都会复制到几台服务器上，当某台故障失效时，生产者和消费者转而使用其它的 Kafka 。

### 架构

![](_imgs/Pasted%20image%2020230704135929.png)

- Producer：消息和数据的生产者，主要负责生产消息到指定的Topic中。
- Topic：同一个Topic包含一个或者多个Partition分区，数据被存储在多个Partition中。
- Partition：Topic物理上的分组，每个Topic包含一个或多个Partition，分区在创建Topic的时候可以指定。分区才是真正存储数据的单元。
- Consumer：消息和数据的消费者，主要负责**主动**到已订阅的Topic中拉取消息并消费。
- ZooKeeper：ZooKeeper负责维护整个Kafka集群的状态，存储Kafka各个节点的信息及状态，实现Kafka集群的高可用，协调Kafka的工作内容。

Kafka集群发布过的消息记录会被持久化到硬盘中，无论该消息是否被消费，发布记录都会被Kafka保留到硬盘当中，可以设置保留期限，过期后被Kafka丢弃以释放空间。


### 分布式协同

每个 Partition 分区都有一个 leader 服务器，假如我们的 Partition1 分区分别被复制到了三台服务器上，其中第二台为这个 Partition 分区的领导者，其它两台服务器都会成为这个 Partition 的followers。

其中 Partition 分片的 leader 处理该 Partition 分区的**所有读和写请求**，而 follower 被动地复制 leader 所发生的改变，如果该 Partition 分片的领导者发生了故障等，两个 follower 中的其中一台服务器将自动成为新的 leader 领导者。每台服务器都充当一些分区的 leader 和一些分区的 follower ，因此集群内的负载非常平衡。

### 使用

1. 下载：[官网](https://kafka.apache.org/) 
2. 启动服务
3. 通过命令行方式体验
4. Java、C++ API

