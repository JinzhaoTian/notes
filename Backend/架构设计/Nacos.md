Nacos（Naming and Configuration Service）是阿里巴巴开源的一个动态服务发现、配置管理和服务管理平台，常用于微服务架构中。

## 核心功能

1. **服务发现与服务健康监测**
2. **动态配置服务**
3. **动态 DNS 服务**
4. **服务及其元数据管理**


## 安装启动

1. **下载与安装**
```bash
# 下载最新稳定版 (当前最新2.2.3)
wget https://github.com/alibaba/nacos/releases/download/2.2.3/nacos-server-2.2.3.tar.gz

# 解压
tar -zxvf nacos-server-2.2.3.tar.gz
cd nacos
```

2. **启动方式**

单机模式启动：
```bash
sh bin/startup.sh -m standalone  # Linux/Mac
cmd startup.cmd -m standalone    # Windows
```

集群模式启动：先配置 `conf/cluster.conf`，然后启动：
```bash
sh bin/startup.sh  # 不带参数
```


## 配置使用

访问地址：
```
http://localhost:8848/nacos
```

默认账号：
```
nacos/nacos
```


1. **创建配置**：
    - Data ID: 通常格式为 `${prefix}-${spring.profiles.active}.${file-extension}`
    - Group: 默认 DEFAULT_GROUP
    - 配置格式: Properties、YAML、JSON等
```yaml
server:
  port: 8080
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test
    username: root
    password: 123456
```


2. **Spring Boot 集成**

配置 `bootstrap.yml`：
```yaml
spring:
  application:
    name: service-name
  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848
        file-extension: yaml
        namespace: dev
        group: DEFAULT_GROUP
```


3. **服务注册**

添加依赖：
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    <version>2022.0.0.0</version>
</dependency>
```

启用服务发现：
```java
@SpringBootApplication
@EnableDiscoveryClient
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

配置服务信息：
```yaml
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
        namespace: dev
        group: DEFAULT_GROUP
        cluster-name: HZ
```


服务发现与调用：
```java
@RestController
public class ConsumerController {
    
    @Autowired
    private DiscoveryClient discoveryClient;
    
    @Autowired
    private RestTemplate restTemplate;
    
    @GetMapping("/call-provider")
    public String callProvider() {
        // 获取服务实例
        List<ServiceInstance> instances = discoveryClient.getInstances("service-provider");
        ServiceInstance instance = instances.get(0);
        
        // 调用服务
        return restTemplate.getForObject(
            "http://" + instance.getHost() + ":" + instance.getPort() + "/hello", 
            String.class);
    }
}
```

## 集群部署

1. 准备3台或更多服务器
2. 每台服务器修改 `conf/cluster.conf`：
```
192.168.1.1:8848
192.168.1.2:8848
192.168.1.3:8848
```

3. 配置数据库（推荐MySQL）
4. 分别启动各个节点