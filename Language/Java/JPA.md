JPA（Java Persistence API）是 Java 平台的一个持久化规范，它为 Java 开发人员提供了一种对象/关系映射（ORM）工具来管理 Java 应用中的关系数据。

### 核心概念

1. **ORM 框架**：JPA 允许开发者通过操作 Java 对象来间接操作数据库表，无需编写繁琐的 SQL 语句。
2. **标准化规范**：JPA 是一套标准接口，具体实现由不同的提供商提供（如 Hibernate、EclipseLink 等）。
3. **JPA 与 Hibernate 关系**：Hibernate 是最流行的 JPA 实现之一，但 JPA 本身只是一个规范。

### 主要特性

- **实体（Entity）**：普通 Java 对象（POJO）通过注解映射到数据库表
- **EntityManager**：核心接口，用于管理实体生命周期和持久化操作
- **JPQL**：Java 持久化查询语言，面向对象的查询语言
- **事务管理**：支持声明式和编程式事务
- **缓存机制**：提供一级和二级缓存提高性能


### 基本示例

```java
@Entity
@Table(name = "employees")
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "emp_name")
    private String name;
    
    // 构造函数、getter和setter
}
```


