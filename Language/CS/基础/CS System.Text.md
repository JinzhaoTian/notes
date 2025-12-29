
## `StringBuilder`

`StringBuilder` 是 `System.Text` 命名空间中的一个类，用于表示可变的字符序列，允许在**不创建新对象**的情况下修改字符串内容。

```csharp
// 低效 - 创建多个字符串对象
string result = "";
for (int i = 0; i < 1000; i++)
{
    result += i.ToString(); // 每次循环都创建新字符串
}

// 高效 - 使用StringBuilder
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
{
    sb.Append(i);
}
string efficientResult = sb.ToString();
```


### 核心特性

1. **内部结构**：`StringBuilder` 内部维护一个字符数组（`char[]`），通过动态调整数组大小来适应内容变化。
```csharp
public class StringBuilder
{
    private char[] _charArray;  // 内部字符数组
    private int _length;         // 当前内容长度
    private int _capacity;       // 数组容量
}
```

2. **容量管理**：
```csharp
// 默认容量为 16 个字符
StringBuilder sb1 = new StringBuilder();

// 指定初始容量
StringBuilder sb2 = new StringBuilder(100); // 初始容量 100

// 从字符串创建，容量会自动调整
StringBuilder sb3 = new StringBuilder("Hello");

// 查看和调整容量
Console.WriteLine(sb3.Capacity); // 当前容量
sb3.EnsureCapacity(200); // 确保容量至少为 200
```

### 常用方法

1. **追加内容（Append）**
```csharp
StringBuilder sb = new StringBuilder();

// 追加字符串
sb.Append("Hello");
sb.Append(" World");

// 追加不同类型
sb.Append(123);          // 整数
sb.Append(45.67);        // 浮点数
sb.Append(true);         // 布尔值
sb.Append(DateTime.Now); // 日期时间

// 追加格式化字符串
sb.AppendFormat(" Value: {0:F2}", 123.456);

// 追加字符
sb.Append('!');

// 追加字符数组
char[] chars = { 'A', 'B', 'C' };
sb.Append(chars);
sb.Append(chars, 1, 2); // 从索引1开始，追加2个字符

Console.WriteLine(sb.ToString()); // "Hello World12345.67True[日期] Value: 123.46!ABCBC"
```

2. **插入内容（Insert）**
```csharp
StringBuilder sb = new StringBuilder("World");

// 在指定位置插入
sb.Insert(0, "Hello ");      // 开头插入
sb.Insert(5, ", ");          // 索引5处插入
sb.Insert(sb.Length, "!");   // 末尾插入

Console.WriteLine(sb.ToString()); // "Hello, World!"
```

3. **删除内容（Remove）**
```csharp
StringBuilder sb = new StringBuilder("Hello, World!");

// 删除指定位置开始的一定数量字符
sb.Remove(5, 2); // 删除从索引5开始的2个字符（", "）

Console.WriteLine(sb.ToString()); // "Hello World!"
```

4. **替换内容（Replace）**
```csharp
StringBuilder sb = new StringBuilder("Hello World World World");

// 替换字符串
sb.Replace("World", "C#"); // 替换所有"World"

// 替换字符
sb.Replace('o', '0'); // 替换所有'o'

// 在指定范围内替换
sb = new StringBuilder("aaaabbbbaaaa");
sb.Replace("a", "A", 0, 4); // 只替换前4个字符中的'a'

Console.WriteLine(sb.ToString()); // "AAAAbbbbaaaa"
```

5. **清空内容（Clear）**
```csharp
StringBuilder sb = new StringBuilder("Some content");
sb.Clear(); // 清空所有内容，Length变为0，Capacity不变
```

### 性能优化

1. **预分配容量**
```csharp
// 如果知道大概大小，预分配容量可以提高性能
int estimatedSize = 1000;
StringBuilder sb = new StringBuilder(estimatedSize);

// 比默认容量自动扩容更高效
```

2. **链式调用**
```csharp
// StringBuilder 的方法返回 this，支持链式调用
string result = new StringBuilder()
    .Append("Name: ").AppendLine("John")
    .Append("Age: ").AppendLine("25")
    .Append("Score: ").Append(95.5)
    .ToString();
```

3. **使用 `AppendJoin`**
```csharp
// 高效连接多个值，带分隔符
var numbers = new int[] { 1, 2, 3, 4, 5 };
StringBuilder sb = new StringBuilder();
sb.AppendJoin(", ", numbers); // "1, 2, 3, 4, 5"
```

