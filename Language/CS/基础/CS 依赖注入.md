
```C#
public class GreetingService
{
	public string Greet(string name) => $"Hello, {name}";
}

public class HomeController
{
	public string Hello(string name)
	{
		var service = new GreetingService();
		return service.Greet(name);
	}
}


static void Main()
{
	var controller = new HomeController();
	string result = controller.Hello("Stephanie");
	Console.WriteLine(result);
}
```


控制反转：
```C#
public interface IGreetingService
{
	string Greet(string name);
}

public class GreetingService: IGreetingService
{
	public string Greet(string) => $"Hello, {name}";
}

public class HomeController
{
	private readonly IGreetingService _greetingService;
	public HomeController(IGreetingService greetingService)
	{
		_greetingService = greetingService ?? throw new ArgumentNullException(nameof(greetingService));
	}

	public string Hello(string name) => _greetingService.Greet(name);
}

static void Main()
{
	var controller = new HomeController(new GreetingService());
	string result = constoller.Hello("world");
	Console.WriteLine(result);
}
```

依赖注入：

使用NuGet包：`Microsoft.Extensions.DependencyInjection`


首先要实例化一个 `ServiceCollection` 对象，该对象具有 `AddSingleton` 和 `AddTransient` 扩展方法，用来注册容器类型。

```C#
static ServiceProvider RegisterServices()
{
	var service = new ServiceCollection();
	service.AddSingleton<IGreetingService, GreetingService>();
	service.AddTransient<HomeController>();
	return services.BuildServiceProvider();
}

static void Main()
{
	using (var container = RegisterServices())
	{
		var controller = container.GetRequiredService<HomeController>();
		string result = controller.Hello("Katharina");
		Console.WriteLine(result);
	}
}

```

