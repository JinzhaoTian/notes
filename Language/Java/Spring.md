[Spring](https://spring.io/) 是一个分层的 Java SE/EE 应用一站式的轻量级开源框架，设计目标是简化企业应用的开发，尤其是在处理复杂的事务、持久化、Web 应用、消息传递等方面提供支持。

**主要优点**：

- 方便解耦，简化开发，通过 Spring 提供的 IoC 容器，我们可以将对象之间的依赖关系交由 Spring 进行控制，避免硬编码造成的程序耦合度高。
- AOP 编程的支持，通过 Spring 提供的 AOP 功能，方便进行面向切面编程。
- 声明式事务的支持，在 Spring 中，我们可以从单调烦闷的事务管理代码中解脱出来，通过声明式方式灵活地进行事务的管理，提高开发效率和质量。
- 方便程序的测试，可以用非容器依赖的编程方式进行几乎所有的测试工作。
- 方便集成各种优秀框架，Spring 提供了对各种优秀框架的直接支持。

# Spring 核心

![](Pasted%20image%2020230707115115.png)
Spring 的核心是 IoC 和 AOP。

## IoC

Spring 的核心概念之一是控制反转（Inversion of Control，IoC），传统的 Java 应用中，应用代码通常会**手动创建对象**并管理它们，而在 Spring 中，Spring 容器负责创建和管理这些对象的生命周期。开发者只需要通过**配置**或者**注解**声明这些依赖关系，Spring 容器会在需要的时候自动注入。

### DI

依赖注入（Dependency Injection，DI），Spring 通过构造器注入、属性注入、方法注入等方式来管理对象之间的依赖关系，从而解耦组件之间的关系。

- **基于注解**：
```java
@Component
public class MyService {
    // ...
}

@Component
public class MyController {
    @Autowired  // 自动注入
    private MyService myService;
}
```

- **基于配置**：
```java
@Configuration
public class AppConfig {
    @Bean
    public MyService myService() {
        return new MyService();
    }
}
```



## AOP

Spring 提供了面向切面编程（Aspect-Oriented Programming，AOP）的支持，使得你可以将日志记录、安全控制、事务管理等横切关注点从业务逻辑中分离出来，减少代码重复。

## Transations

Spring 提供了统一的事务管理，支持声明式事务和编程式事务管理，它能够管理不同类型的事务（如 JDBC 事务、JTA 事务等），使得应用程序能够更加简洁和可靠地处理事务。


# Spring Modules

Spring 模块（Spring Modules）是 Spring 核心项目（Spring Framework）中的**基础功能组件**，提供特定领域的核心能力，通常需要手动组合和配置。

- **Spring Core**：核心容器，提供 IoC 容器、依赖注入（DI）、Bean 管理等核心功能。
- **Spring Context**：上下文配置，提供企业服务
- **Spring AOP**：面向切面编程支持（如日志、事务管理）
- **Spring DAO**：数据访问抽象
- **Spring ORM**：对象关系映射集成，整合 Hibernate、JPA 等 ORM 框架。
- **Spring Web**：Web 应用支持
- **Spring MVC**：模型-视图-控制器实现

**特点**：

- **轻量级**：每个模块解决一个具体问题（如 IoC、AOP、数据访问等）。
- **可插拔**：按需引入，不强制依赖其他模块。
- **需显式配置**：通常通过 XML 或 Java Config 配置。


# Spring 框架

Spring 框架（Spring Frameworks）是基于 Spring 模块构建的**高层解决方案**，通常整合多个模块并提供**开箱即用**的功能，用于解决更复杂的场景问题。


- [**Spring Boot**](Spring%20Boot.md)：快速构建生产级应用，内嵌服务器、自动配置、Starter依赖简化依赖管理。
- [**Spring Cloud**](Spring%20Cloud.md)：微服务架构支持（服务发现、配置中心、熔断器等）。
- **Spring Security**：认证和授权框架（OAuth2、JWT、ACL等）。
- **Spring Batch**：批处理框架（大数据量定时任务处理）。
- **Spring Integration**：企业集成模式（消息通道、路由、适配器等）。


**特点**：

- **集成性**：组合多个Spring模块并添加额外功能（如自动配置、默认约定）。
- **简化开发**：减少配置，提供“约定优于配置”的体验。
- **独立使用**：通常作为独立项目存在（如Spring Boot、Spring Cloud）。


| **需求**      | **推荐框架**                   |
| ----------- | -------------------------- |
| 快速开发单体应用    | Spring Boot + Spring MVC   |
| 微服务架构       | Spring Boot + Spring Cloud |
| 高并发响应式应用    | Spring WebFlux + Reactor   |
| 批处理任务       | Spring Batch               |
| 安全管控        | Spring Security            |
| 灵活的数据查询 API | Spring GraphQL             |


# Spring 常用注解


- **`@Component`** ：标记为 Spring 组件
- **`@Service`** ：标记为服务层组件
- **`@Repository`** ：标记为数据访问层组件
- **`@Controller`** ：标记为控制器组件
- **`@RestController`** ：组合了 **`@Controller`** 和 **`@ResponseBody`**
- **`@Autowired`** ：自动注入依赖
- **`@Value`** ：注入属性值
- **`@Configuration`** ：标记为配置类
- **`@Bean`** ：声明一个bean
- **`@RequestMapping`** ：映射web请求
- **`@GetMapping`** / **`@PostMapping`** 等：特定 HTTP 方法的映射