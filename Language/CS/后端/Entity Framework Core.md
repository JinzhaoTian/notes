Entity Framework Core 是微软推出的一个轻量级、可扩展、开源且跨平台的[ORM](../../../Database/Database/ORM.md)框架，是经典 Entity Framework 的现代化版本。

## 关键特性

1. **跨平台支持**
	- 支持 Windows、Linux 和 macOS
	- 与 .NET Core/.NET 5+ 完全兼容

2. **数据库提供程序**
	- 支持多种数据库：
	    - SQL Server/SQL Azure (`Microsoft.EntityFrameworkCore.SqlServer`)
	    - SQLite (`Microsoft.EntityFrameworkCore.Sqlite`)
	    - PostgreSQL (`Npgsql.EntityFrameworkCore.PostgreSQL`)
	    - MySQL (`Pomelo.EntityFrameworkCore.MySql`)
	    - Oracle (`Oracle.EntityFrameworkCore`)
	    - Cosmos DB (`Microsoft.EntityFrameworkCore.Cosmos`)
	    - 内存数据库 (用于测试，`Microsoft.EntityFrameworkCore.InMemory`)

3. **迁移系统**
	- 代码优先迁移：通过 C# 代码管理数据库架构变更
	- 命令行工具：`dotnet ef migrations add` 和 `dotnet ef database update`

4. **高性能**
	- 查询优化
	- 批量操作
	- 延迟加载（可选）
	- 显式加载
	- 变更跟踪优化


## 主要组件

- **DbContext**：数据库会话，负责与数据库交互
- **DbSet**：代表数据库中的表
- **实体类**：普通的 C# 类，映射到数据库表
- **LINQ 提供程序**：将 LINQ 查询转换为 SQL 查询

### DbContext

DbContext 是 Entity Framework Core（EF Core）中的核心类，它代表与数据库的会话，用于查询和保存数据到数据库。

```cs
public class ApplicationDbContext : DbContext
{
    // 构造函数
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    // 定义实体集合(DbSet属性)
    public DbSet<Customer> Customers { get; set; }
    public DbSet<Order> Orders { get; set; }

    // 可选的模型配置
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // 在这里配置模型关系、约束等
    }
}
```


### DbSet

`DbSet<TEntity>` 代表数据库中的表，每个实体类型（模型类）通常对应一个 `DbSet` ，用于查询和保存该类型的实例。



### 导航属性

导航属性是 EF Core 模型中用于表示实体之间关系的属性，就像一条“导航路径”，让你能够从一个实体方便地访问与其相关联的其他实体。

导航属性主要分为两类：

1. **引用导航属性**：表示“一对一”或“多对一”关系中的单一实体。
2. **集合导航属性**：表示“一对多”或“多对多”关系中的实体集合。

```cs
public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }

    // 集合导航属性：一个博客有多篇文章
    public List<Post> Posts { get; set; }
}

public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }

    // 外键属性 (指向 Blog 的 BlogId)
    public int BlogId { get; set; }

    // 引用导航属性：一篇文章属于一个博客
    public Blog Blog { get; set; }
}
```
在这个例子中：
- `Blog.Posts`：是一个**集合导航属性**。
- `Post.Blog`：是一个**引用导航属性**。
- `Post.BlogId`：是一个**外键属性**，它存储了关系的目标主键值。


#### 核心价值

1. **面向对象的编程体验**：你可以使用像 `myPost.Blog.Url` 或 `myBlog.Posts.Select(p => p.Title)` 这样的直观代码来访问相关数据，而不需要手动编写复杂的 JOIN 查询。
2. **延迟加载与预先加载**：EF Core 可以利用导航属性来延迟加载相关数据（当第一次访问该属性时）或通过 `Include` 方法预先加载。

> [!tip] 没有外键的关系表也能建立导航属性吗？
> 可以，数据库中的外键约束和 EF Core 中的导航属性是两个不同层面的概念。
> - **数据库外键**：是数据库层面用于强制引用完整性的约束。它确保数据的一致性。
> - **EF Core 导航属性**：是代码层面用于描述对象间关联的元数据。它指导 EF Core 如何生成查询和管理关系。





## 使用示例

1. **定义模型**：
```cs
public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }
    
    // 导航属性
    public List<Post> Posts { get; set; }
}

public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }
    
    // 外键
    public int BlogId { get; set; }
    // 导航属性
    public Blog Blog { get; set; }
}
```

2. **创建 DbContext**
```cs
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder options)
        => options.UseSqlServer("Your_Connection_String");
}
```

3. **使用迁移创建数据库**
```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

4. **基本 CRUD 操作**
```cs
// 创建
using (var db = new BloggingContext())
{
    var blog = new Blog { Url = "http://example.com" };
    db.Blogs.Add(blog);
    db.SaveChanges();
}

// 读取
using (var db = new BloggingContext())
{
    var blogs = db.Blogs
        .Where(b => b.Url.Contains("example"))
        .ToList();
}

// 更新
using (var db = new BloggingContext())
{
    var blog = db.Blogs.Find(1);
    blog.Url = "http://updated.com";
    db.SaveChanges();
}

// 删除
using (var db = new BloggingContext())
{
    var blog = db.Blogs.Find(1);
    db.Blogs.Remove(blog);
    db.SaveChanges();
}
```


### 列名和模型字段的映射

1. **使用数据注解（Data Annotations）**：
```cs
[Table("db_blog_table")]
public class Blog
{
	[Column("blog_id")]
    public int BlogId { get; set; }
    [Column("url")]
    public string Url { get; set; }
    
    // 导航属性
    public List<Post> Posts { get; set; }
}
```

2. **使用 Fluent API**（推荐方式）
```cs
using Microsoft.EntityFrameworkCore;

public class YourDbContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Blog>(entity =>
        {
            entity.ToTable("db_blog_table");
            
            entity.Property(e => e.BlogId)
	              .HasColumnName("blog_id");
                
            entity.Property(e => e.Url)
	              .HasColumnName("url")
	              .HasMaxLenghth(100);
        });
    }
}
```


3. **使用配置类**：
```cs
public class BlogConfiguration : IEntityTypeConfiguration<Blog>
{
    public void Configure(EntityTypeBuilder<Blog> builder)
    {
        builder.ToTable("db_blog_table");
        builder.Property(e => e.BlogId)
		       .HasColumnName("blog_id");
		builder.Property(e => e.Url)
		       .HasColumnName("url")
		       .HasMaxLenghth(100);
        // ... 其他配置
    }
}

// 在 DbContext 中应用
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.ApplyConfiguration(new BlogConfiguration());
}
```

> [!tip] 不同类型的配置
> `HasMaxLength` 只适用于字符串类型，对于 `datetime`、`int`、`float` 等数值和日期类型，不需要也不应该使用 `HasMaxLength`。
> 1. **字符串类型（`string`）**
> 	- 指定最大长度：`.HasMaxLength(50)`
> 	- 对于配置最大长度（`nvarchar(max)`）：
> 		- 不配置，默认就是 `nvarchar(max)`
> 		- 显式使用 `.HasMaxLength(-1)`
> 		- 使用 `.HasColumnType("nvarchar(max)")`
> 2. **日期时间类型（`DateTime`）**
> 	- 指定精度：`.HasPrecision(3)`，指定小数秒精度
> 3. **整数类型（`int`，`long`，`short`）**
> 	- 指定数据库类型： `.HasColumnType("int")`
> 4. **浮点类型（`float`，`double`，`decimal`）**
> 	- 使用 `.HasPrecision` 和 `.HasScale` 指定精度
> 	- `double` 默认映射为 `float(53)`
> 	- `float` 默认映射为 `real`，即 `float(24)`

### 联合查询

EF Core 可以使用 LINQ 进行联合查询，最常用的方式是使用 `Join` 方法。

```cs
// 定义视图对应的实体类
public class ViewBookDetail
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Author { get; set; }
    public string CateName { get; set; }
}

public class ViewCateStats
{
    public string Name { get; set; }
    public int BookCount { get; set; }
    public int AuthorCount { get; set; }
}

// 在DbContext中注册
public class MyDbContext : DbContext
{
    public DbSet<ViewBookDetail> ViewBookDetails { get; set; }
    public DbSet<ViewCateStats> ViewCateStats { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // 配置为无键实体并映射到视图
        modelBuilder.Entity<ViewBookDetail>(entity =>
        {
            entity.HasNoKey(); // 标记为无键
            entity.ToView("View_BookDetails"); // 映射到视图名称
        });

        modelBuilder.Entity<ViewCateStats>(entity =>
        {
            entity.HasNoKey();
            entity.ToView("View_CateStats");
        });
    }
}
```


```cs
// 示例：基于分类名称连接两个视图
var query = from book in dbContext.ViewBookDetails
            join stat in dbContext.ViewCateStats 
            on book.CateName equals stat.Name
            select new
            {
                BookName = book.Name,
                Author = book.Author,
                CateName = book.CateName,
                BookCountInCate = stat.BookCount,
                AuthorCountInCate = stat.AuthorCount
            };

var result = query.ToList();
```


或者使用方法语法：

```cs
var query = dbContext.ViewBookDetails
    .Join(dbContext.ViewCateStats,
          book => book.CateName,
          stat => stat.Name,
          (book, stat) => new
          {
              BookName = book.Name,
              Author = book.Author,
              CateName = book.CateName,
              BookCountInCate = stat.BookCount,
              AuthorCountInCate = stat.AuthorCount
          });
```





## 高级特性

1. **关系配置**
```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>()
                .HasMany(b => b.Posts)
                .WithOne(p => p.Blog)
                .HasForeignKey(p => p.BlogId)
                .OnDelete(DeleteBehavior.Cascade);
}
```

2. **复杂查询**
```cs
var blogsWithPosts = context.Blogs
                            .Include(b => b.Posts) // 显式加载关联数据
                            .Where(b => b.Posts.Any(p => p.Title.Contains("EF")))
                            .OrderBy(b => b.Url)
                            .ToList();
```

3. **原生 SQL 查询**
```cs
var blogs = context.Blogs
                   .FromSqlRaw("SELECT * FROM Blogs WHERE Url LIKE '%dotnet%'")
                   .ToList();
```

4. **全局查询过滤器**
```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>()
                .HasQueryFilter(b => !b.IsDeleted);
}
```




## 连接建立

Entity Framework Core 中建立数据库连接的时机

### 连接建立时机

1. **首次执行数据库操作时**（延迟连接）
    - 当您第一次调用需要数据库连接的操作时（如 `ToList()`、`FirstOrDefault()`、`SaveChanges()` 等）
    - 这是最常见的情况，EF Core 采用"按需连接"策略

2. **调用 `OpenConnection()` 时**

3. **调用 `EnsureCreated()` 或 `EnsureDeleted()` 时**


### 连接生命周期

- 连接通常会在操作完成后关闭（除非使用连接池）
- 默认情况下，EF Core 使用连接池，所以物理连接可能会被 ADO.NET 连接池保留

### 重要注意事项

- DbContext 的构造函数**不会**立即建立数据库连接
- 连接是轻量级的，EF Core 设计为只在需要时才建立连接
- 您可以通过 `DbContext.Database.GetDbConnection().State` 检查当前连接状态

### 连接建立流程

以这个方式为例：
```cs
// 注册
public static IServiceCollection AddDatabaseServices(this IServiceCollection services, 
													IConfiguration configuration)
{
	// 添加 DB 数据库上下文
	services.AddDbContext<ApplicationDBContext>(options =>
		options.UseSqlServer(configuration.GetConnectionString("DefaultConnection")));

	services.AddScoped<IDBService, DBService>();

	return services;
}
```

```cs
// 注入
public class DBService : IDBService
{
	private readonly ApplicationDBContext _dbContext;

	public DBService(ApplicationDBContext dbContext)
	{
		_dbContext = dbContext;
	}

	public List<Project> GetUserProjects(string userId)
	{
		var projects = _dbContext.Projects
								 .Where(c => c.UserId == userId)
								 .Distinct()
								 .ToList();
		return projects;
	}

	// ...

}
```

1. **依赖注入容器注册**：
    - `AddDbContext` 默认以 Scoped 生命周期注册 `ApplicationDBContext`
    - 配置了使用 SQL Server 和连接字符串，但此时**尚未建立实际数据库连接**

2. **服务解析时**：
    - 当请求到达（如 ASP.NET Core 的 HTTP 请求）时，容器会解析 `IDBService`
    - 因为 `DBService` 需要 `ApplicationDBContext`，容器会先创建 `ApplicationDBContext` 实例
    - 此时**仍然没有建立实际数据库连接**，只是创建了上下文对象

3. **实际连接建立时机**：
    - 当 `DBService` 中的方法**第一次执行需要数据库的操作**时（如查询、保存等），才会建立实际连接
```cs
var users = _dbContext.Users.ToList();  // 这里才会真正建立连接
```

