AMQP（Advanced Message Queuing Protocol，高级消息队列协议）是一个进程间传递异步消息的网络协议。


运行原理：

发布者（Publisher）发布消息（Message），经由交换机（Exchange），交换机根据路由规则将收到的消息分发给与该交换机绑定的队列（Queue），最后 AMQP 代理会将消息投递给订阅了此队列的消费者，或者消费者按照需求自行获取。

