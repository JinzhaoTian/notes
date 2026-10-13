Attribute（特性）是 C# 中把**声明式元数据**附加到程序元素上的机制,继承自 `System.Attribute` 的类，使用时写在方括号里，例如：

```csharp
[Obsolete("请使用 NewMethod")]
public void OldMethod() { }
```

`Obsolete` 实际对应 `ObsoleteAttribute`。

> [!tip] C# 约定
> 特性类名以 `Attribute` 结尾，使用时可以省略后缀。

## Attribute 是什么

Attribute 是继承自 `System.Attribute` 的类，**用来将元数据编译后存储在程序集中**，可以应用于程序集、模块、类、结构、方法、属性、字段、参数、返回值、泛型参数等。

```csharp
[assembly: AssemblyVersion("1.0.0.0")]

[Serializable]
public class User
{
    [Required]
    public string Name { get; set; }

    [Obsolete]
    public void Old() { }
}
```

Attribute 参数类型有限制，通常只能是：
- 基本类型：`int`、`bool`、`string`、`double` 等
- `Type`
- 枚举
- 上述类型的一维数组
- 不能是任意对象、委托、普通类实例

## 语法

自定义特性常用 `AttributeUsage` 修饰：

```csharp
[AttributeUsage(
    AttributeTargets.Class | AttributeTargets.Method,
    AllowMultiple = true,
    Inherited = true)]
public sealed class AuthorAttribute : Attribute { }
```

- `AttributeTargets`：能加在哪里。
- `AllowMultiple`：能否重复添加，默认 `false`。
- `Inherited`：子类/重写方法是否继承该特性，默认 `true`。




## 核心原理



### 编译期：写入元数据

```csharp
[Author("Alice", Version = 1)]
public class Demo { }
```

1. 找到 `AuthorAttribute` 类，确认它继承 `System.Attribute`。
2. 检查 `AttributeUsage`，确认能加在 `class` 上。
3. 检查构造参数和命名参数类型是否合法。
4. 把这条特性编码进程序集的元数据表 `CustomAttribute` 中。


### 运行期：通过反射读取并实例化

Attribute 本身不会自动执行。只有被读取时，才会创建实例。

例如，
```csharp
[AttributeUsage(AttributeTargets.Class, AllowMultiple = true, Inherited = true)]
public sealed class AuthorAttribute : Attribute
{
    public string Name { get; }
    public int Version { get; set; }
    public AuthorAttribute(string name)
    {
        Name = name;
    }
}

[Author("Alice", Version = 1)]
[Author("Bob", Version = 2)]
public class Demo { }
```

如果想要读取，可以：
```csharp
using System.Reflection;

var attrs = typeof(Demo).GetCustomAttributes<AuthorAttribute>();

foreach (var a in attrs)
{
    Console.WriteLine($"{a.Name}, {a.Version}");
}
```

**如果没人读取，Attribute 就只是元数据，不会执行任何逻辑**。



## 用途

1. **给编译器指令**
```csharp
[Obsolete("不要用了")]
[Conditional("DEBUG")]
[CallerMemberName]
```

2. **给运行时/CLR 指令**
```csharp
[DllImport("user32.dll")]
[StructLayout(LayoutKind.Sequential)]
[MarshalAs(UnmanagedType.LPStr)]
[Serializable]
```

3. **框架通过反射读取，实现声明式配置**
	- ASP.NET Core：`[HttpGet]`、`[Route]`、`[Authorize]`
	- EF Core：`[Key]`、`[Required]`、`[Column]`
	- JSON：`[JsonPropertyName]`
	- 验证：`[Required]`、`[StringLength]`
	- 测试：`[Fact]`、`[Theory]`、`[Test]`
	- Swagger/OpenAPI：`[ProducesResponseType]`

4. **源生成器 / 代码生成**
	- `System.Text.Json` 源生成
	- `Regex` 源生成
	- `LoggerMessage`
	- 各种 ORM、Mapper、DI 容器

5. **AOP / 拦截 / 权限 / 审计**：C# 本身不原生支持 AOP，但很多框架会读取 Attribute，然后生成代理、织入 IL 或生成代码，实现：
	- 日志
	- 缓存
	- 事务
	- 权限检查
	- 审计

6. **静态分析 / 工具**：分析器可以读取 Attribute，例如 `[NotNull]`、`[Pure]`，辅助代码检查。


