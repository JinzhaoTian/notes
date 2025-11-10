Fluent Validation 是一个流行的 .NET 验证库，它提供了一种流畅的接口（fluent interface）来为你的模型定义强类型的验证规则。在 [ASP.NET Core](ASP.NET%20Core.md) 中，它可以很好地集成到 MVC 的验证管道中。

## 主要特点

1. **流畅的接口**：使用链式方法调用定义验证规则，代码可读性高
2. **强类型**：验证规则与模型属性直接关联，编译时检查
3. **可重用**：验证逻辑可以封装在单独的验证器类中
4. **可扩展**：支持自定义验证规则和错误消息
5. **测试友好**：验证逻辑易于单元测试



## 使用示例

1. **安装 NuGet 包**
```bash
dotnet add package FluentValidation.AspNetCore
```

2. **创建验证器类**
```cs
public class CustomerValidator : AbstractValidator<Customer>
{
    public CustomerValidator()
    {
        RuleFor(customer => customer.Name)
            .NotEmpty().WithMessage("姓名不能为空")
            .Length(2, 100).WithMessage("姓名长度必须在2到100个字符之间");
            
        RuleFor(customer => customer.Email)
            .NotEmpty().WithMessage("邮箱不能为空")
            .EmailAddress().WithMessage("邮箱格式不正确");
            
        RuleFor(customer => customer.Age)
            .GreaterThan(0).WithMessage("年龄必须大于0")
            .LessThan(150).WithMessage("年龄必须小于150");
    }
}
```

3. **在 ASP.NET Core 中注册**：在 `Startup.cs` 或 `Program.cs` 中
```cs
// 添加 FluentValidation 服务
builder.Services.AddFluentValidation(fv => 
{
    fv.RegisterValidatorsFromAssemblyContaining<CustomerValidator>();
    // 或者自动注册所有验证器
    // fv.RegisterValidatorsFromAssembly(Assembly.GetExecutingAssembly());
});

// 如果你使用 MVC
builder.Services.AddControllersWithViews()
    .AddFluentValidation();
```

4. **在 Controller 中使用**
```cs
[HttpPost]
public IActionResult Create(Customer customer)
{
    if (!ModelState.IsValid)
    {
        // 验证失败，返回错误
        return BadRequest(ModelState);
    }
    
    // 验证通过，处理业务逻辑
    return Ok();
}
```


## 高级功能

1. **条件验证**：
```csharp
RuleFor(customer => customer.Phone)
        .NotEmpty()
        .When(customer => customer.PhoneRequired)
        .WithMessage("电话是必填项");
```

2. **集合验证**：
```csharp
RuleForEach(customer => customer.Orders)
        .SetValidator(new OrderValidator());
```

3. **自定义验证**：
```csharp
RuleFor(customer => customer.Name)
	.Must(name => name.Contains(" "))
	.WithMessage("姓名必须包含空格");
```

4. **异步验证**：
```csharp
RuleFor(customer => customer.Email)
	.MustAsync(async (email, cancellation) => 
		await IsEmailUniqueAsync(email))
	.WithMessage("邮箱已被注册");
```

5. **级联模式**：
```csharp
RuleFor(customer => customer.Name)
	.Cascade(CascadeMode.Stop)
	.NotEmpty()
	.Length(2, 100);
```