AutoMapper 是一个流行的对象到对象映射库，用于在 .NET 应用程序中自动将一个对象转换为另一个对象。在 [ASP.NET Core](ASP.NET%20Core.md) 中，它特别有用，因为它可以简化领域模型和视图模型（或 DTOs）之间的转换。

## 主要功能

1. **自动映射**：根据约定自动映射同名属性
2. **自定义映射**：可以配置特定属性的映射规则
3. **嵌套映射**：支持复杂对象的嵌套映射
4. **集合映射**：支持集合类型（List、Array 等）的映射
5. **条件映射**：可以设置条件来决定是否映射某个属性

## 使用示例

1. **安装 NuGet 包**
```bash
dotnet add package AutoMapper
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
```

2. **基本配置**：在 `Startup.cs` 或程序入口文件中：
```cs
services.AddAutoMapper(typeof(Startup)); // 或者 Assembly
```

3. **创建映射配置**
```cs
public class AutoMapperProfile : Profile
{
    public AutoMapperProfile()
    {
        // 简单映射
        CreateMap<User, UserDto>();
        
        // 自定义映射
        CreateMap<Order, OrderDto>()
            .ForMember(dest => dest.OrderDate, 
                       opt => opt.MapFrom(src => src.CreatedAt.ToString("d")));
    }
}
```

4. **使用映射**
```cs
public class UserController : Controller
{
    private readonly IMapper _mapper;
    
    public UserController(IMapper mapper)
    {
        _mapper = mapper;
    }
    
    public IActionResult GetUser(int id)
    {
        var user = _userRepository.GetById(id);
        var userDto = _mapper.Map<UserDto>(user);
        return Ok(userDto);
    }
}
```


## 高级用法

1. **反向映射**：
```cs
CreateMap<User, UserDto>().ReverseMap();
```

2. **忽略属性**：
```cs
CreateMap<User, UserDto>()
    .ForMember(dest => dest.Password, opt => opt.Ignore());
```

3. **条件映射**：
```cs
CreateMap<User, UserDto>()
    .ForMember(dest => dest.Email, 
               opt => opt.Condition(src => !string.IsNullOrEmpty(src.Email)));
```

4. **使用值转换器**：
```cs
CreateMap<DateTime, string>().ConvertUsing(dt => dt.ToString("yyyy-MM-dd"));
```

