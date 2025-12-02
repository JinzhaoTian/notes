Spring Boot 是一个用于简化 Spring 应用初始搭建和开发过程的框架，是一个项目脚手架，解决如何快速启动一个完整应用（包含 Web 、数据、安全等）的问题，整合了如 Spring Data、Spring Security、Spring Cloud 等**垂直领域的专业工具项目**。

现代开发中，90% 的场景会**以 Spring Boot 为底座**，按需添加其他模块。

## 使用

#### 环境准备

- **Java JDK**：推荐 JDK 8 或更高版本
- **构建工具**：Maven 或 Gradle
- **IDE**：IntelliJ IDEA、Eclipse 或 VS Code

#### 创建项目

1. **方式一**：使用 [Spring Initializr](https://start.spring.io/) 
	- 选择构建工具（Maven/Gradle）
	- 选择 Spring Boot 版本
	- 添加项目元数据（Group，Artifact）
	- 添加所需依赖（如 Web，JPA 等）
	- 生成并下载项目

2. **方式二**：使用 IDE 集成


#### 项目结构

典型 Spring Boot 项目结构：
```java
src/
├── main/
│   ├── java/
│   │   └── com/example/yourapp/
│   │       ├── YourApplication.java (主启动类)
│   │       ├── controller/ (控制器)
│   │       ├── service/ (服务层)
│   │       ├── repository/ (数据访问)
│   │       └── model/ (实体类)
│   └── resources/
│       ├── static/ (静态资源)
│       ├── templates/ (模板文件)
│       └── application.properties (配置文件)
└── test/ (测试代码)
```


#### 核心注解

- **`@SpringBootApplication`**：主类注解，
	- 组合了 `@Configuration`, `@EnableAutoConfiguration` 和 `@ComponentScan` 
- **`@RestController`**：声明 RESTful 控制器
- `@RequestMapping` / `@GetMapping` / `@PostMapping`等：定义请求映射
- **`@Service`**：业务逻辑层组件
- **`@Repository`**：数据访问层组件
- **`@Autowired`**：依赖注入


#### 示例

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @Autowired
    private ProductService productService;

    @GetMapping
    public List<Product> getAllProducts() {
        return productService.getAllProducts();
    }

    @GetMapping("/{id}")
    public Product getProductById(@PathVariable Long id) {
        return productService.getProductById(id);
    }

    @PostMapping
    public Product createProduct(@RequestBody Product product) {
        return productService.saveProduct(product);
    }
    
    // 其他CRUD操作...
}
```


#### 配置数据库连接

在 `application.properties` 或 `application.yml` 中配置：
```java
# MySQL配置示例
spring.datasource.url=jdbc:mysql://localhost:3306/yourdb
spring.datasource.username=root
spring.datasource.password=yourpassword
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA配置
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5Dialect
```


#### 使用 Spring Data JPA

```java
@Entity
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private Double price;
    // getters and setters...
}

@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    // 自定义查询方法
    List<Product> findByPriceLessThan(Double price);
}
```

#### 运行

**IDE**：

- 直接运行主类中的 `main` 方法

**Maven**：

- 使用 Maven 命令：`mvn spring-boot:run`
- 打包为可执行 Jar ：`mvn package`，然后运行 `java -jar target/your-app.jar`

**使用包装器（推荐）**

- 如果项目包含 Maven Wrapper（Spring Initializr 默认会生成），最好使用
```bash
./mvnw spring-boot:run

mvnw.cmd spring-boot:run  # Windows
```



## 功能扩展

- **安全**：集成 Spring Security
- **缓存**：使用 Spring Cache 抽象
- **消息队列**：集成 RabbitMQ 或 Kafka
- **监控**：使用 Spring Boot Actuator
- **测试**：使用 `@SpringBootTest` 进行集成测试


## 配置

Spring Boot 支持多种配置方式：
- `application.properties` 或 `application.yml`
- 环境特定配置：`application-{profile}.properties`
- 使用 `@ConfigurationProperties` 绑定配置到 Java 对象