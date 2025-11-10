Spring Cloud 是一套基于 [Spring Boot](Spring%20Boot.md) 的**微服务开发工具集**，它为分布式系统开发提供了全面的解决方案，简化了微服务架构中的常见模式实现。

Spring Cloud 主要提供以下功能：

1. **服务发现与注册**：如 Eureka、Consul、[ZooKeeper](../../Backend/分布式中间件/ZooKeeper.md)
2. **客户端负载均衡**：Ribbon
3. **声明式 REST 客户端**：Feign
4. **API 网关**：Spring Cloud Gateway
5. **分布式配置管理**：Spring Cloud Config
6. **熔断器模式**：Hystrix（已逐步被 Resilience4j 替代）
7. **分布式跟踪**：Sleuth 与 Zipkin 集成
8. **消息驱动**：Spring Cloud Stream


### 主要组件

- **Spring Cloud Netflix**：集成 Netflix OSS 组件（Eureka, Hystrix, Zuul 等）
- **Spring Cloud Alibaba**：集成阿里巴巴的微服务解决方案（Nacos, Sentinel 等）
- **Spring Cloud Gateway**：API 网关服务
- **Spring Cloud Config**：分布式配置中心
- **Spring Cloud Bus**：消息总线，用于配置变更的传播
- **Spring Cloud Sleuth**：分布式请求链路跟踪


#### Spring Cloud Alibaba

Spring Cloud Alibaba 提供了 [Nacos](../../Backend/架构设计/Nacos.md) 作为配置中心，下面详细介绍如何使用 Nacos 进行分布式配置管理。

##### 环境准备

1. 安装 Nacos Server
	- 从 [Nacos 官网](https://nacos.io/) 下载最新版本
	- 解压后运行启动命令：
	    - Linux/Unix/Mac ： `sh startup.sh -m standalone`
	    - Windows ： `cmd startup.cmd -m standalone`
	- 访问控制台：[http://localhost:8848/nacos](http://localhost:8848/nacos)（默认账号/密码：nacos/nacos）

2. 在项目的 `pom.xml` 中添加 Spring Cloud Alibaba Nacos Config 依赖：
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    <version>2022.0.0.0</version> <!-- 根据Spring Cloud版本选择 -->
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    <version>2022.0.0.0</version>
</dependency>
```

##### 基本配置使用

1. **创建 `bootstrap.yml`** ：Spring Cloud Alibaba 配置需要在 `bootstrap.yml` 中配置
```yaml
spring:
  application:
    name: user-service
  profiles:
    active: dev
  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848
        file-extension: yaml
        namespace: dev
        group: DEFAULT_GROUP
      discovery:
        server-addr: 127.0.0.1:8848
```

2. 在 Nacos 中添加配置
	- 登录 Nacos 控制台
	- 进入 "配置管理" → "配置列表"
	- 点击 "+" 新建配置：
	    - Data ID： `user-service-dev.yaml` 
		    - 格式：`spring.application.name−spring.application.name−{spring.profiles.active}.${file-extension}`
	    - Group ： `DEFAULT_GROUP`
	    - 配置格式 ：YAML    
	    - 配置内容示例：
```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/user_db
    username: root
    password: 123456

custom:
  config: hello-nacos
```

3. 在代码中使用
```java
@RestController
@RefreshScope // 支持动态刷新
public class ConfigController {
    
    @Value("${custom.config}")
    private String config;
    
    @GetMapping("/config")
    public String getConfig() {
        return config;
    }
}
```


##### 高级功能

1. 多环境配置
	- 创建不同环境的配置：
	    - 开发环境: `user-service-dev.yaml`
	    - 测试环境: `user-service-test.yaml`
	    - 生产环境: `user-service-prod.yaml`
	- 通过 `spring.profiles.active` 指定环境
    

2. 共享配置
	- 创建共享配置 `common-config.yaml`
    - 在 `bootstrap.yml` 中配置：
```yaml
spring:
  cloud:
    nacos:
      config:
        shared-configs:
          - data-id: common-config.yaml
            group: DEFAULT_GROUP
            refresh: true
```


3. 配置优先级：Spring Cloud Alibaba Nacos Config 配置优先级
	- 应用名-环境.yaml (最高优先级)
	- 应用名.yaml
	- 共享配置
	- 本地配置 (最低优先级)

4. 动态配置刷新
	- 使用 `@RefreshScope` 注解
	- 在 Nacos 控制台修改配置并发布
	- 应用会自动获取最新配置


##### 示例项目结构

```
src/main/
├── java
│   └── com.example
│       └── UserServiceApplication.java
└── resources
    ├── application.yml
    └── bootstrap.yml
```

