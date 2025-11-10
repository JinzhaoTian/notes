RabbitMQ 是一个开源的、实现了 [AMQP](AMQP.md) 的完整可复用企业消息系统，主要用于在分布式系统中**异步传递消息**，能在不同应用程序或组件之间可靠地传输数据，有效实现系统解耦、流量削峰等重要功能。并且支持多种开发语言，Java、C#、Python、Node.js、C/C++ 等。

![](imgs/Pasted%20image%2020240715174602.png)

## 安装

### Docker

```bash
docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3.13-management
```

### Linux

1. **安装**：
```bash
sudo apt install rabbitmq-server
```

2. **启动服务**：
```bash
sudo systemctl start rabbitmq-server
sudo systemctl enable rabbitmq-server
```

3. **创建用户**：默认的 `guest` 用户通常只允许本地连接，建议创建新用户
```bash
# 创建用户
sudo rabbitmqctl add_user myuser mypassword
# 设置用户权限（在默认的"/"虚拟主机下）
sudo rabbitmqctl set_permissions -p / myuser ".*" ".*" ".*"
```

4. **启用管理插件**：这个插件会提供一个 Web 管理界面，非常方便
```bash
sudo rabbitmq-plugins enable rabbitmq_management
```


### Windows

RabbitMQ 服务端代码是使用并发式语言 Erlang 编写的，安装 Rabbit MQ 的前提是安装 Erlang。

1. **RabbitMQ 与 Erlang/OTP 的版本对照表**：![](imgs/Pasted%20image%2020251103154953.png)

2. **[安装 Erlang](../../Language/Erlang.md#安装)**

3. **安装 RabbitMQ**：
	- 下载地址：[Installing on Windows | RabbitMQ](https://www.rabbitmq.com/docs/install-windows)
	- 点击安装
	- **启动服务**
	- 打开主页：[http://127.0.0.1:15672/](http://127.0.0.1:15672/)，默认账号：guest，默认密码：guest![](imgs/Pasted%20image%2020251103160700.png)![](imgs/Pasted%20image%2020251103160635.png)
	- **创建用户**：![](imgs/Pasted%20image%2020251103162353.png)




## 使用案例

### 案例：使用消息队列进行流量削峰填谷

1. **添加依赖**
```xml
<!-- Spring Boot Starter for AMQP -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

2. **配置生产者**
```Java
@Configuration
public class RabbitMQConfig {
    
    @Bean
    public Queue peakClippingQueue() {
        return new Queue("peak.clipping.queue", true); // 持久化队列
    }
    
    @Bean
    public Exchange peakClippingExchange() {
        return new DirectExchange("peak.clipping.exchange");
    }
    
    @Bean
    public Binding binding() {
        return BindingBuilder
				.bind(peakClippingQueue())
				.to(peakClippingExchange())
				.with("peak.clipping.routingKey")
				.noargs();
    }
}
```

3. **发送消息服务**
```Java
@Service
public class MessageProducer {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public void sendMessage(Object message) {
        // 添加生产限流，当队列积压超过阈值时减慢生产速度
        rabbitTemplate.convertAndSend(
            "peak.clipping.exchange", 
            "peak.clipping.routingKey", 
            message,
            m -> {
                m.getMessageProperties()
				 .setDeliveryMode(MessageDeliveryMode.PERSISTENT);
                return m;
            });
    }
}
```

4. **配置消费者**
```Java
@Configuration
public class ConsumerConfig {
    
    @Bean
    public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory(ConnectionFactory connectionFactory) {
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        factory.setConcurrentConsumers(5); // 初始消费者数量
        factory.setMaxConcurrentConsumers(20); // 最大消费者数量
        factory.setPrefetchCount(50); // 每个消费者预取消息数
        return factory;
    }
    
}
```

5. **消息监听处理**
```Java
@Service
public class MessageConsumer {
    
    private static final Logger logger = LoggerFactory.getLogger(MessageConsumer.class);
    
    @RabbitListener(queues = "peak.clipping.queue")
    public void handleMessage(Object message, Channel channel, 
            @Header(AmqpHeaders.DELIVERY_TAG) long tag) {
        try {
            // 实际业务处理
            processMessage(message);
            
            // 手动确认消息
            channel.basicAck(tag, false);
        } catch (Exception e) {
            logger.error("处理消息失败", e);
            try {
                // 处理失败，放入死信队列
                channel.basicNack(tag, false, false);
            } catch (IOException ex) {
                logger.error("消息NACK失败", ex);
            }
        }
    }
    
    private void processMessage(Object message) {
        // 这里实现具体的业务逻辑
        // 模拟处理耗时
        try {
            Thread.sleep(100); // 模拟处理时间
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

6. **监控队列积压**
```Java
@Service
public class QueueMonitor {
    
    @Autowired
    private RabbitAdmin rabbitAdmin;
    
    @Scheduled(fixedRate = 5000) // 每5秒检查一次
    public void monitorQueue() {
        QueueInformation queueInfo = rabbitAdmin.getQueueInfo("peak.clipping.queue");
        if (queueInfo != null) {
            int messageCount = queueInfo.getMessageCount();
            adjustConsumers(messageCount);
        }
    }
    
    private void adjustConsumers(int pendingMessages) {
        // 根据积压消息数动态调整消费者数量
        if (pendingMessages > 1000) {
            scaleUpConsumers();
        } else if (pendingMessages < 100) {
            scaleDownConsumers();
        }
    }
    
    private void scaleUpConsumers() {
        // 实际实现中可以通过Kubernetes或云服务API增加消费者实例
        // 或者调整Spring的concurrentConsumers参数
    }
    
    private void scaleDownConsumers() {
        // 减少消费者实例
    }
}
```



