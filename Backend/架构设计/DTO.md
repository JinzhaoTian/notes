DTO（Data Transfer Object，数据传输对象）是一种设计模式，主要用于在不同层或系统之间**高效、安全地传输数据**。它的核心目的是**封装数据**，减少通信开销，并避免暴露内部实现细节。

## 主要特点

1. **纯数据结构**
    - 只包含属性（字段）和简单的 `getter/setter` 方法，没有业务逻辑。
    - 示例（Java）：
```java
public class UserDTO {
	private String name;
	private String email;
	// getter 和 setter 方法
}
```

2. **跨层/跨系统传输**
    - 用于 Controller 层返回给前端、微服务间 API 调用、分布式系统通信等场景。

3. **与领域模型解耦**
    - DTO 通常与数据库实体（Entity）分离，避免直接暴露敏感字段（如密码、ID 等）。
    - 示例：UserEntity（数据库模型）可能包含 20 个字段，但 UserDTO 只返回前端需要的5个字段。

4. **扁平化结构**
    - 可能合并多个领域对象的数据，简化传输。例如：`OrderDTO` 可能包含用户信息（User）和订单详情（Order）。


## 使用原因

- **性能优化**：减少不必要的数据传输（如排除敏感字段或无用字段）。
- **安全性**：隐藏内部数据结构，避免泄露敏感信息。
- **版本兼容性**：独立演化API和数据层，不影响客户端。
- **适配不同需求**：同一实体可根据场景生成不同的 DTO（如`UserSimpleDTO`和`UserDetailDTO`）。


## 使用场景

1. **前后端交互**
    - 后端API返回给前端的JSON通常对应一个DTO。
    - 示例（Spring Boot）：
```java
@GetMapping("/users/{id}")
public UserDTO getUser(@PathVariable Long id) {
	User user = userService.findById(id);
	return convertToDTO(user); // 将User实体转换为UserDTO
}
```

2. **微服务间调用**
    - 服务 A 通过 REST/gRPC 调用服务 B 时，通过 DTO 传递数据。

3. **避免循环引用**
    - 如实体间有关联关系（如 `Order` 关联 `User` ），直接序列化可能导致无限循环。DTO 可以切断这种关联。


## 注意事项

- **避免过度使用**：简单场景可直接使用实体，无需额外封装。
- **工具简化**：使用 MapStruct 、ModelMapper 等库简化 Entity 到 DTO 的转换。
- **不可变性**：考虑将 DTO 设计为不可变对象（如用 `final` 字段 + 构造方法）。

通过 DTO ，你能更灵活、安全地控制数据的传输和暴露范围。