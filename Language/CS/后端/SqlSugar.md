SqlSugar 是一款轻量级的国产 ORM（对象关系映射）框架，用于 .NET 平台，主要简化数据库操作。

## 主要特点

1. **简单易用**：API 设计简洁，学习成本低
2. **高性能**：相比其他 ORM 框架有较好的性能表现
3. **多数据库支持**：支持 SQL Server、MySQL、Oracle、PostgreSQL、SQLite 等
4. **丰富的功能**：包括分页、事务、读写分离、AOP 等
5. **代码先行/数据库先行**：支持两种开发模式


## 用法示例

```cs
// 创建实体类
public class Student
{
    [SugarColumn(IsPrimaryKey = true, IsIdentity = true)]
    public int Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
}

// 配置连接
var db = new SqlSugarScope(new ConnectionConfig()
{
    ConnectionString = "server=.;database=test;uid=sa;pwd=123",
    DbType = DbType.SqlServer,
    IsAutoCloseConnection = true
});

// 查询示例
var list = db.Queryable<Student>().Where(it => it.Age > 18).ToList();

// 插入示例
db.Insertable(new Student() { Name = "张三", Age = 20 }).ExecuteCommand();
```


# CRUD

### Queryable

```C#
var getAll = db.Queryable<Student>().ToList();
var getAllNoLock = db.Queryable<Student>().With(SqlWith.NoLock).ToList();
```

### Updateable

```C#
//update reutrn Update Count
var t1= db.Updateable(updateObj).ExecuteCommand();

//Only  update  Name 
var t3 = db.Updateable(updateObj).UpdateColumns(it => new { it.Name }).ExecuteCommand();

//Ignore  Name and TestId
var t4 = db.Updateable(updateObj).IgnoreColumns(it => new { it.Name, it.TestId }).ExecuteCommand();

//update List<T>
var t7 = db.Updateable(updateObjs).ExecuteCommand();

//Where By Expression
var t9 = db.Updateable(it => new class() { name = "a",createtime = p }).Where(it => it.Id == 1).ExecuteCommand();
```

### Insertable

```C#
//Insert reutrn Insert Count
var t2 = db.Insertable(insertObj).ExecuteCommand();

//Insert reutrn Identity Value
var t3 = db.Insertable(insertObj).ExecuteReutrnIdentity();

//Only  insert  Name 
var t4 = db.Insertable(insertObj).InsertColumns(it => new { it.Name, it.SchoolId }).ExecuteReutrnIdentity();

//Ignore TestId
var t5 = db.Insertable(insertObj).IgnoreColumns(it => new { it.Name, it.TestId }).ExecuteReutrnIdentity();

//Insert List<T>
var s9 = db.Insertable(insertObjs).InsertColumns(it => new { it.Name }).ExecuteCommand();
```


### Deleteable

```C#
//by entity
db.Deleteable<Student>().Where(new Student() { Id = 1 }).ExecuteCommand();

//by primary key
db.Deleteable<Student>().In(1).ExecuteCommand();

//by primary key array
db.Deleteable<Student>().In(new int[] { 1, 2 }).ExecuteCommand();

//by expression
db.Deleteable<Student>().Where(it => it.Id == 1).ExecuteCommand();
```

### SqlQueryable

```C#
var list = db.SqlQueryable<Student>("select * from student").ToPageList(1, 2);

var list2 = db.SqlQueryable<Student>("select * from student").Where(it => it.Id == 1).ToPageList(1, 2);

var list3 = db.SqlQueryable<Student>("select * from student").Where("id=@id", new { id = 1 }).ToPageList(1, 2);
```


### SaveQueues

```C#
var db = GetInstance();
db.Insertable<Student>(new Student() { Name = "a" }).AddQueue();
db.Insertable<Student>(new Student() { Name = "b" }).AddQueue();
db.SaveQueues();

db.Insertable<Student>(new Student() { Name = "a" }).AddQueue();
db.Insertable<Student>(new Student() { Name = "b" }).AddQueue();
db.Insertable<Student>(new Student() { Name = "c" }).AddQueue();
db.Insertable<Student>(new Student() { Name = "d" }).AddQueue();
var ar = db.SaveQueuesAsync();

db.Queryable<Student>().AddQueue();
db.Queryable<School>().AddQueue();
db.AddQueue("select * from student where id=@id", new { id = 1 });
var result2 = db.SaveQueues<Student, School, Student>();  
```

### Saveable

```csharp
db.Saveable<Student>(entity).ExecuteReturnEntity();
db.Saveable<Student>(new Student() { Name = "" })
                  .InsertColumns(it => it.Name)
                  .UpdateColumns(it => new { it.Name, it.CreateTime }
                  .ExecuteReturnEntity();
```


