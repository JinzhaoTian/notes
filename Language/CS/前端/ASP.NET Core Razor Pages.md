ASP.NET Core Razor Pages 是 ASP.NET Core 中一种以页面为中心的 Web 开发模型，用来构建服务端渲染的 Web UI。它建立在 ASP.NET Core MVC 基础之上，使用 Razor 语法（C# + HTML），但把“一个页面的视图和处理逻辑”放在一起，而不是像 MVC 那样由 Controller 集中处理多个页面。

## 核心组成

一个典型的 Razor Page 通常包含两个文件：

- `Index.cshtml`：页面视图，使用 Razor 语法。
- `Index.cshtml.cs`：页面模型 `PageModel`，处理请求逻辑。

例如：
```html
@page
@model IndexModel
<h1>@Model.Message</h1>
<form method="post">
    <input asp-for="Name" />
    <button type="submit">提交</button>
</form>
```

```csharp
public class IndexModel : PageModel
{
    [BindProperty]
    public string Name { get; set; }
    public string Message { get; set; }
    public void OnGet()
    {
        Message = "你好";
    }
    public void OnPost()
    {
        Message = $"你好，{Name}";
    }
}
```

在 `Program.cs` 中启用：
```csharp
builder.Services.AddRazorPages();
app.MapRazorPages();
```

## 工作方式

1. `.cshtml` 文件顶部的 `@page` 指令使它成为一个 Razor Page，而不是普通 MVC 视图。
2. 默认路由基于文件路径：
    - `Pages/Index.cshtml` → `/` 或 `/Index`
    - `Pages/Products/Edit.cshtml` → `/Products/Edit`
3. GET 请求执行 `OnGet`，POST 请求执行 `OnPost`。
4. 也可以有 `OnGetAsync`、`OnPostAsync`，或自定义处理器如 `OnPostDelete`。
5. 支持模型绑定、验证、依赖注入、过滤器、Tag Helpers、布局页等，与 MVC 共享同一套基础设施。

