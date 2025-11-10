
## 基本规范

1. **控制器（Controller）**：
    - 接收 HTTP 请求并返回 HTTP 响应。
    - 处理路由、模型验证、身份验证和授权。
    - 调用服务层处理业务逻辑，并将结果返回给客户端。
    - 不应包含业务逻辑，只负责协调请求和响应。
2. **服务层（Service）**：
    - 包含业务逻辑和应用程序规则。
    - 处理数据访问层（通过仓储或其他数据访问模式）来获取和持久化数据。
    - 服务层可以被控制器调用，也可以被其他服务调用。
    - 服务层应依赖于抽象（接口）而不是具体实现，以便于测试和扩展。
3. **模型（Model）**：
    - 实体模型（Entity Models）：代表数据库中的表，用于数据持久化。
    - 数据传输对象（DTOs）：用于在客户端和服务器之间传输数据，通常与实体模型分开，以避免直接暴露内部数据结构。
    - 视图模型（View Models）：用于在控制器和视图之间传递数据（在 API 项目中通常不使用，因为 API 通常返回 JSON，所以常用 DTOs）。
4. **数据访问层（Data Access Layer）**：
    - 负责与数据库交互，可以使用 Entity Framework Core 等 ORM。
    - 通常使用仓储模式（Repository Pattern）来抽象数据访问，使服务层不直接依赖于数据访问技术。
5. **API 设计**：
    - 使用 RESTful 风格设计 API 端点。
    - 使用 HTTP 动词（GET、POST、PUT、DELETE 等）表示操作。
    - 使用合适的 HTTP 状态码表示结果（如200成功，400客户端错误，500服务器错误等）。
    - 使用版本控制（如 URL 版本控制）来管理 API 变更。

## 示例

### 分层架构

```
ProjectName/
├── ProjectName.API/           # 表现层
├── ProjectName.Application/   # 应用层
├── ProjectName.Domain/        # 领域层
├── ProjectName.Infrastructure/# 基础设施层
└── ProjectName.Common/        # 公共组件
```

### 职责

1. **Model（Domain Models）**
	- 定义核心业务实体
	- 封装业务逻辑和验证规则
	- 保持与数据库无关的纯净领域模型
```cs
// ProjectName.Domain/Entities/User.cs
public class User : EntityBase
{
    public string Username { get; private set; }
    public string Email { get; private set; }
    public UserStatus Status { get; private set; }
    public DateTime CreatedAt { get; private set; }
    
    // 私有构造函数用于EF Core
    private User() { }
    
    public User(string username, string email)
    {
        Username = username ?? throw new ArgumentNullException(nameof(username));
        Email = email ?? throw new ArgumentNullException(nameof(email));
        Status = UserStatus.Active;
        CreatedAt = DateTime.UtcNow;
        
        // 领域验证
        if (!IsValidEmail(email))
            throw new DomainException("Invalid email format");
    }
    
    public void Deactivate()
    {
        Status = UserStatus.Inactive;
    }
    
    private static bool IsValidEmail(string email)
    {
        return Regex.IsMatch(email, @"^[^@\s]+@[^@\s]+\.[^@\s]+$");
    }
}

// 值对象
public class Address : ValueObject
{
    public string Street { get; }
    public string City { get; }
    public string ZipCode { get; }
    
    protected override IEnumerable<object> GetEqualityComponents()
    {
        yield return Street;
        yield return City;
        yield return ZipCode;
    }
}
```

2. **DTOs（Data Transfer Objects）**
	- 数据传输对象，用于 API 输入输出
	- 不包含业务逻辑
```cs
// ProjectName.Application/DTOs/UserDtos.cs
public class UserDto
{
    public Guid Id { get; set; }
    public string Username { get; set; }
    public string Email { get; set; }
    public string Status { get; set; }
    public DateTime CreatedAt { get; set; }
}

public class CreateUserRequest
{
    [Required]
    [StringLength(50)]
    public string Username { get; set; }
    
    [Required]
    [EmailAddress]
    public string Email { get; set; }
    
    [Required]
    [MinLength(6)]
    public string Password { get; set; }
}

public class UpdateUserRequest
{
    [EmailAddress]
    public string? Email { get; set; }
}
```

3. **Service（Application Layer）**
	- 协调领域对象完成用例
	- 事务管理
	- 输入验证和输出映射
	- 不包含核心业务逻辑
```cs
// ProjectName.Application/Interfaces/IUserService.cs
public interface IUserService
{
    Task<Result<UserDto>> GetUserByIdAsync(Guid id);
    Task<Result<UserDto>> CreateUserAsync(CreateUserRequest request);
    Task<Result<bool>> UpdateUserAsync(Guid id, UpdateUserRequest request);
    Task<Result<bool>> DeleteUserAsync(Guid id);
    Task<PaginatedResult<UserDto>> GetUsersAsync(UserQuery query);
}

// ProjectName.Application/Services/UserService.cs
public class UserService : IUserService
{
    private readonly IUserRepository _userRepository;
    private readonly IEmailService _emailService;
    private readonly IMapper _mapper;
    private readonly ILogger<UserService> _logger;

    public UserService(
        IUserRepository userRepository,
        IEmailService emailService,
        IMapper mapper,
        ILogger<UserService> logger)
    {
        _userRepository = userRepository;
        _emailService = emailService;
        _mapper = mapper;
        _logger = logger;
    }

    public async Task<Result<UserDto>> CreateUserAsync(CreateUserRequest request)
    {
        try
        {
            // 检查用户是否存在
            if (await _userRepository.ExistsByEmailAsync(request.Email))
                return Result<UserDto>.Failure("User with this email already exists");

            // 创建领域对象
            var user = new User(request.Username, request.Email);
            
            // 保存到数据库
            await _userRepository.AddAsync(user);
            await _userRepository.UnitOfWork.SaveChangesAsync();

            // 发送欢迎邮件
            await _emailService.SendWelcomeEmailAsync(user.Email, user.Username);

            _logger.LogInformation("User created with ID: {UserId}", user.Id);
            
            return Result<UserDto>.Success(_mapper.Map<UserDto>(user));
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error creating user with email: {Email}", request.Email);
            return Result<UserDto>.Failure("Failed to create user");
        }
    }

    public async Task<PaginatedResult<UserDto>> GetUsersAsync(UserQuery query)
    {
        var specification = new UserSpecification(query);
        var users = await _userRepository.GetPaginatedAsync(
            specification, 
            query.PageNumber, 
            query.PageSize);
            
        return _mapper.Map<PaginatedResult<UserDto>>(users);
    }
}
```


4. **Controller（API Layer）**
	- HTTP 请求处理
	- 身份认证和授权
	- 输入模型验证
	- 响应格式统一处理
	- 异常处理
```cs
// ProjectName.API/Controllers/UsersController.cs
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("1.0")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;
    private readonly IMapper _mapper;

    public UsersController(IUserService userService, IMapper mapper)
    {
        _userService = userService;
        _mapper = mapper;
    }

    [HttpGet("{id:guid}")]
    [Authorize(Policy = "RequireUserRead")]
    [ProducesResponseType(typeof(ApiResponse<UserDto>), StatusCodes.Status200OK)]
    [ProducesResponseType(typeof(ApiResponse), StatusCodes.Status404NotFound)]
    public async Task<IActionResult> GetUser(Guid id)
    {
        var result = await _userService.GetUserByIdAsync(id);
        
        if (!result.Succeeded)
            return NotFound(ApiResponse.Error(result.Message));

        return Ok(ApiResponse.Success(result.Data));
    }

    [HttpPost]
    [AllowAnonymous]
    [ProducesResponseType(typeof(ApiResponse<UserDto>), StatusCodes.Status201Created)]
    [ProducesResponseType(typeof(ApiResponse), StatusCodes.Status400BadRequest)]
    public async Task<IActionResult> CreateUser([FromBody] CreateUserRequest request)
    {
        if (!ModelState.IsValid)
            return BadRequest(ApiResponse.Error(ModelState));

        var result = await _userService.CreateUserAsync(request);
        
        if (!result.Succeeded)
            return BadRequest(ApiResponse.Error(result.Message));

        return CreatedAtAction(
            nameof(GetUser), 
            new { id = result.Data.Id }, 
            ApiResponse.Success(result.Data));
    }

    [HttpPut("{id:guid}")]
    [Authorize(Policy = "RequireUserWrite")]
    [ProducesResponseType(typeof(ApiResponse), StatusCodes.Status200OK)]
    [ProducesResponseType(typeof(ApiResponse), StatusCodes.Status400BadRequest)]
    public async Task<IActionResult> UpdateUser(Guid id, [FromBody] UpdateUserRequest request)
    {
        if (!ModelState.IsValid)
            return BadRequest(ApiResponse.Error(ModelState));

        var result = await _userService.UpdateUserAsync(id, request);
        
        if (!result.Succeeded)
            return BadRequest(ApiResponse.Error(result.Message));

        return Ok(ApiResponse.Success(result.Data));
    }

    [HttpGet]
    [Authorize(Policy = "RequireUserRead")]
    [ProducesResponseType(typeof(ApiResponse<PaginatedResult<UserDto>>), StatusCodes.Status200OK)]
    public async Task<IActionResult> GetUsers([FromQuery] UserQuery query)
    {
        var result = await _userService.GetUsersAsync(query);
        return Ok(ApiResponse.Success(result));
    }
}
```


### API 设计规范

1. **RESTful 设计原则**
```cs
// 统一响应格式
public class ApiResponse<T>
{
    public bool Success { get; set; }
    public string Message { get; set; }
    public T Data { get; set; }
    public DateTime Timestamp { get; set; } = DateTime.UtcNow;

    public static ApiResponse<T> Success(T data, string message = null) => new()
    {
        Success = true,
        Message = message,
        Data = data
    };
}

public class ApiResponse : ApiResponse<object>
{
    public static ApiResponse Error(string message) => new()
    {
        Success = false,
        Message = message
    };
}

// 分页响应
public class PaginatedResult<T>
{
    public List<T> Items { get; set; }
    public int PageNumber { get; set; }
    public int PageSize { get; set; }
    public int TotalCount { get; set; }
    public int TotalPages => (int)Math.Ceiling(TotalCount / (double)PageSize);
}
```

2. **路由规范**
```cs
// 好的路由设计
[Route("api/v{version:apiVersion}/users")]
[Route("api/v{version:apiVersion}/products/{productId}/reviews")]

// 避免的路由设计
[Route("api/getAllUsers")]
[Route("api/updateUserInfo")]
```

3. **HTTP 方法使用**

| 操作   | HTTP 方法 | 路由                 | 描述     |
| ---- | ------- | ------------------ | ------ |
| 获取列表 | GET     | /api/v1/users      | 获取用户列表 |
| 获取单个 | GET     | /api/v1/users/{id} | 获取特定用户 |
| 创建   | POST    | /api/v1/users      | 创建新用户  |
| 更新   | PUT     | /api/v1/users/{id} | 全量更新用户 |
| 部分更新 | PATCH   | /api/v1/users/{id} | 部分更新用户 |
| 删除   | DELETE  | /api/v1/users/{id} | 删除用户   |

### 依赖注入

```cs
// ProjectName.API/Program.cs 或 Startup.cs
public static class DependencyInjection
{
    public static IServiceCollection AddApplication(this IServiceCollection services)
    {
        // 注册服务
        services.AddScoped<IUserService, UserService>();
        services.AddScoped<IProductService, ProductService>();
        
        // 注册仓储
        services.AddScoped<IUserRepository, UserRepository>();
        
        // AutoMapper
        services.AddAutoMapper(Assembly.GetExecutingAssembly());
        
        // MediatR (如果使用CQRS)
        services.AddMediatR(Assembly.GetExecutingAssembly());
        
        // FluentValidation
        services.AddValidatorsFromAssembly(Assembly.GetExecutingAssembly());
        
        return services;
    }
}
```


### 异常处理中间件

```cs
public class ExceptionHandlingMiddleware
{
    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        try
        {
            await next(context);
        }
        catch (DomainException ex)
        {
            context.Response.StatusCode = StatusCodes.Status400BadRequest;
            await context.Response.WriteAsJsonAsync(ApiResponse.Error(ex.Message));
        }
        catch (NotFoundException ex)
        {
            context.Response.StatusCode = StatusCodes.Status404NotFound;
            await context.Response.WriteAsJsonAsync(ApiResponse.Error(ex.Message));
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "An unhandled exception has occurred");
            context.Response.StatusCode = StatusCodes.Status500InternalServerError;
            await context.Response.WriteAsJsonAsync(ApiResponse.Error("An internal server error occurred"));
        }
    }
}
```


