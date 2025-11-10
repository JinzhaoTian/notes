面向对象三个概念：继承、封装和多态。


## 基本面向对象

### 继承

- 对于类，C# 支持单重继承，不支持多重继承，支持多层继承。
- 对于接口，C# 支持单重和多重继承。
- 对于结构体，C# 不支持继承，结构体自动派生自 `System.ValueType` 。



### 多态

使用多态性，可以动态调用方法，而不是在编译期间定义。编译器创建一个虚拟方法表，其中列出了可以在运行期间调用的方法。

可以使用 `base` 关键字调用基类方法。



### 接口

声明接口在语法上与声明抽象类完全相同，但不允许提供接口中任何成员的实现方式。一般情况下，接口只能包含**方法**、**属性**、索引器和事件的声明。



## 泛型面向对象


### 接口约束

```C#
public partial class KdTree<T>
        where T : class
{
}
```

其中，`where` 修饰的语法代表接口约束，有若干种接口约束类型。
![](_imgs/Pasted%20image%2020230801171219.png)


### 扩展方法

扩展方法是 C# 3.0 引入的新特性，使用它，可以在不修改某一类的代码的情况下，实现该类方法的扩展。这是因为很多库都是以 dll 的形式组织的，没有办法去修改这些。

为一个类添加扩展方法，需要三个要素：
1. 扩展方法所在的类为静态类
2. 扩展方法本身要为静态方法
3. 扩展方法的第一个参数要用关键字 `this` ，指向要扩展的类。

代码示例：
```C#
namespace ThCADExtension
{
    public static class ThRectangleExtension
    {
        // ...
        
        public static AcRectangle FlattenRectangle(this AcRectangle rectangle)
        {
            Plane XYPlane = new Plane(Point3d.Origin, Vector3d.ZAxis);
            Matrix3d matrix = Matrix3d.Projection(XYPlane, XYPlane.Normal);
            return GetTransformedRectangle(rectangle, matrix);
        }
        
		// ...
    }
}

```

使用方法：
```C#
namespace ThCADExtension
{
    public static class ThBlockReferenceExtensions
    {
	    // ...
		
		public static Polyline ToOBB(this BlockReference br, Matrix3d ecs2Wcs)
        {
            using (var acadDatabase = AcadDatabase.Use(br.Database))
            {
                var blockTableRecord = acadDatabase.Blocks.Element(br.BlockTableRecord);
                var rectangle = blockTableRecord.GeometricExtents().ToRectangle();
                return rectangle.GetTransformedRectangle(ecs2Wcs).FlattenRectangle();
            }
        }
		
		// ...
    }
}
```


### 分部方法

在C#中，可以使用 `partial` 关键字拆分多个.cs文件中的类、结构、方法或接口的实现。编译程序时，编译器将合并来自多个.cs文件的所有实现。
```C#
// EmployeeProps.cs
public partial class Employee
{
    public int EmpId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public int Age { get; set; }
}

// EmployeeMethods.cs
public partial class MyPartialClass
{
    public Employee(int Id, string name)
    {
        this.EmpId = Id;
        this.Name = name;
    }

    public void DisplayEmployeeInfo()
    {
        Console.WriteLine(this.EmpId + " " this.FirstName + " " + this.LastName);
    }

    public void Save(int id, string firstName, string lastName)
    {
        Console.WriteLine("Saved!");
    }
}
```
上述两个分开的类文件，在编译时合并成一个类文件：
```C#
public class Employee
{
    public int EmpId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public int Age { get; set; }

    public Employee(int Id, string name)
    {
        this.EmpId = Id;
        this.Name = name;
    }

    public void DisplayEmployeeInfo()
    {
        Console.WriteLine(this.EmpId + " " this.FirstName + " " + this.LastName);
    }

    public void Save(int id, string firstName, string lastName)
    {
        Console.WriteLine("Saved!");
    }
}
```

对于分部类：
- **所有分部类定义必须位于同一程序集和命名空间中**。
- 所有分部都必须具有相同的可访问性，例如公共或私有等。
- 如果任何分部声明为抽象、密封或基类型，那么整个类声明为相同的类型。
- 不同的分部可以具有不同的基本类型，因此最终类将继承所有基本类型。
- **Partial修饰符只能出现在关键字class，struct 或 interface之前**。
- 允许嵌套分部类型。


### 委托

方法当做参数来传递的话，就要用到委托。简单来说**委托是一个类型**，这个类型可以赋值一个方法的引用。类似于C++的函数指针，只不过是类型安全、面向对象的。


## 反射和特性


### 特性 Attribute

C#代码中：
```C#
[Serializable]
public class STRtree<TItem> : AbstractSTRtree<Envelope, TItem>, ISpatialIndex<TItem>
{
}
```
`[Serializable]` 是Python/TypeScript中的装饰器吗？其实不是一个东西，这种语法在C#中称之为特性（Attribute）。

特性、装饰器，其实都是设计模式中的**装饰器模式**，同时也是AOP（Aspect Oriented Programming，面向切面编程）思想。AOP把系统分解为不同的关注点，或者称之为切面（Aspect），是一种在运行时，**动态地将代码切入到类的指定方法、指定位置上**的编程思想。

C#中的特性（Attribute）是一个类，继承自 `Attribute` 类，然后可以包含任意你想要的属性字段。
```C#
namespace System
{
    [AttributeUsage(AttributeTargets.Class | AttributeTargets.Struct | AttributeTargets.Enum | AttributeTargets.Delegate, Inherited = false)]
    [ComVisible(true)]
    public sealed class SerializableAttribute : Attribute
    {   // 特性使用时可以省略后面的"Attribute"，所以SerializableAttribute和Serializable是同个东西
        internal static Attribute GetCustomAttribute(RuntimeType type)
        {
            // ...
            return new SerializableAttribute();
        }

        internal static bool IsDefined(RuntimeType type)
        {
            return type.IsSerializable;
        }
    }
}
```

Python里的装饰器（Decorator）是一个语法糖，用以美化装饰器的写法，其本质上是一个有逻辑的，可以执行的函数。Python的装饰器只是普通函数调用的另一种写法，等价于自己嵌套调用。
```Python
def fuck(check=lambda x:True):
	def _(func):
		def _(*args,**kwargs):
			return check(func(*args,**kwargs)) 
		return _
	return _ 

def check(x):
	return x>10

@fuck(lambda x:x>10)
def mark_1(args):
	return 1+args
	
@fuck(check)
def mark_2(args):
	return 1+args

@fuck()
def mark_3(args):
	return 1+args

def mark_4(args):
	return 1+args 
	
print(mark_1(10))
print(mark_2(10))
print(mark_3(10))
print(fuck(lambda x:x>1)(mark_4)(10))
print(fuck(check)(mark_4)(10))
```
以上五个输出中的调用完全等价。


Java的注解（Annotation）实际上是给语法元素打一个标记。比如你可以给一个函数打一个标记，给一个类打一个标记等等。Java只保证记录这个标记，但是不会主动根据这给标记做任何事。如在Spring里，给一个私有成员打 `@Autowired` 这个标记：
```Java
public class XXXService {
   @Autowired
   private XXXXRepository xxxxRepository;
   
   // ...
}
```
**如果不用Spring框架的话，不会有任何事情发生**，直接访问这个字段就是空。当如果你配置了合适的处理流程，而这个流程就会根据有没有这个标记干活。比如你要求Spring “Auto Scan” 并且注入依赖，这个处理过程会用反射去读哪些元素被做了某个特定标记。没有标记就不理，有标记就注入。
