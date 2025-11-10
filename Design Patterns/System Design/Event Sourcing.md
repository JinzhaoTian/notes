Event Sourcing（事件溯源）是一种数据持久化模式，它通过记录**状态改变的事件**（而非最终状态）来存储和重建应用程序的状态。

**核心思想**：**系统的状态由一系列事件（Event）的累积决定**，而不是直接存储当前状态。

![](../System%20Design/imgs/Pasted%20image%2020250704160224.png)


## **核心概念**

1. **事件（Event）**
    - 表示系统中发生的**事实**（Fact），是不可变的（Immutable）。
    - 例如：`OrderCreated`、`PaymentProcessed`、`UserAddressUpdated`。
    - 通常包含：事件ID、时间戳、相关聚合（Aggregate）ID、事件数据。

2. **事件存储（Event Store）**
    - 专门存储事件的数据库（如`EventStoreDB`、Kafka、Cassandra等）。
    - 事件一旦存储，就不能修改或删除（可追加新事件来纠正错误）。

3. **聚合（Aggregate）**
    - 领域驱动设计（DDD）中的概念，代表业务实体（如`Order`、`User`）。
    - 通过应用（Apply）事件来改变状态。

4. **投影（Projection）**
    - 从事件流中生成**查询优化的视图**（如SQL表、文档数据库）。
    - 例如：`OrderSummary` 表由 `OrderCreated`、`OrderPaid` 等事件生成。

5. **快照（Snapshot）**
    - 优化手段，避免每次重建聚合时重放所有事件。
    - 定期保存聚合的当前状态，后续只需重放快照之后的事件。

## **如何工作？**

1. **写操作（Command）**
    - 用户执行一个命令（如`PlaceOrder`）。
    - 系统生成一个事件（如`OrderCreated`）并存入`Event Store`。
    - 事件被应用到聚合，更新其状态。

2. **读操作（Query）**
    - 直接查询**投影（Projection）**（如SQL表），而不是从事件流重建。
    - 如果投影不存在，可以重放事件流重新计算。

3. **重建状态**
    - 通过重放所有相关事件（如`OrderCreated` → `OrderPaid` → `OrderShipped`）重建聚合的当前状态。


