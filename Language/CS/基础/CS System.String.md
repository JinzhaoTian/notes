C# 中的字符串是 `System.String` 类（关键字 `string`）在内存中**以 UTF-16 编码存储**，表示一个不可变的 Unicode 字符序列，是 .NET 中最常用的数据类型之一。

字符串在 C# 中是不可变的（immutable），这意味着一旦创建，字符串的内容就不能更改。当对字符串进行修改时，实际上是创建了一个新的字符串对象。

```csharp
// string 是 System.String 的别名
string str1 = "Hello";
System.String str2 = "World"; // 效果相同
```

## 原理

1. **字符串是引用类型**：
```csharp
// 字符串存储在堆中，变量存储的是引用
string str1 = "Hello";
string str2 = str1; // str2引用同一块内存
```

2. **字符串的不可变性**（Immutability）：
```csharp
// 字符串一旦创建就不能修改
string s = "Hello";
s = s + " World"; // 创建新字符串，原字符串不变

// 内存示意
// 原始: "Hello" 在堆中
// 修改后: "Hello World" 是新对象
```

3. **字符串驻留**（String Interning）：.NET 会自动驻留字符串字面量，相同的字符串字面量会引用相同的内存位置。
```csharp
// .NET会缓存字符串字面量
string a = "Hello";
string b = "Hello";
bool areSame = object.ReferenceEquals(a, b); // true

// 手动驻留
string c = "Hel" + "lo";
bool areSame2 = object.ReferenceEquals(a, c); // true（编译时优化）

string d = new string("Hello".ToCharArray());
bool areSame3 = object.ReferenceEquals(a, d); // false
string e = string.Intern(d); // 手动驻留
```

## 使用

### 创建

1. **字面量方式**：
```csharp
string str1 = "Hello, World!";
```

2. **使用 String 构造函数**：
```csharp
// 从字符数组创建
char[] letters = { 'H', 'e', 'l', 'l', 'o' };
string str2 = new string(letters);

// 重复字符
string str3 = new string('*', 10); // "**********"
```

3. **字符串插值**（C# 6.0+，.NET Framework 4.6+，.NET Core 1.0+）：
```csharp
string name = "John";
int age = 25;
string message = $"My name is {name} and I'm {age} years old.";
```

4. **逐字字符串**（`@`）：
```csharp
// 普通字符串需要转义
string path1 = "C:\\Users\\John\\Documents";

// 逐字字符串不需要转义
string path2 = @"C:\Users\John\Documents";

// 逐字字符串可以包含换行
string multiline = @"Line 1
Line 2
Line 3";
```

### 操作

1. **连接字符串**：
```csharp
// 使用+运算符
string concat1 = "Hello" + " " + "World";

// 使用Concat方法
string concat2 = string.Concat("Hello", " ", "World");

// 使用StringBuilder（频繁修改时推荐）
StringBuilder sb = new StringBuilder();
sb.Append("Hello");
sb.Append(" ");
sb.Append("World");
string result = sb.ToString();
```

2. **字符串比较**：
```csharp
string str1 = "Hello";
string str2 = "HELLO";

// 区分大小写比较
bool equal1 = str1 == str2; // false
bool equal2 = str1.Equals(str2); // false

// 不区分大小写比较
bool equal3 = str1.Equals(str2, StringComparison.OrdinalIgnoreCase); // true
bool equal4 = string.Equals(str1, str2, StringComparison.OrdinalIgnoreCase); // true

// Compare方法
int result = string.Compare(str1, str2, StringComparison.OrdinalIgnoreCase); // 0表示相等
```

3. **查找和提取**：
```csharp
string text = "Hello, World!";

// 查找
int index1 = text.IndexOf("World"); // 7
int index2 = text.LastIndexOf("o"); // 8

// 检查开头和结尾
bool starts = text.StartsWith("Hello"); // true
bool ends = text.EndsWith("!"); // true

// 提取子字符串
string sub1 = text.Substring(7); // "World!"
string sub2 = text.Substring(7, 5); // "World"

// 分割
string[] parts = text.Split(','); // ["Hello", " World!"]
```

4. **修改字符串**：
```csharp
string text = "Hello, World!";

// 替换
string replaced = text.Replace("World", "C#"); // "Hello, C#!"

// 大小写转换
string upper = text.ToUpper(); // "HELLO, WORLD!"
string lower = text.ToLower(); // "hello, world!"

// 去除空白
string spaced = "  Hello  ";
string trimmed = spaced.Trim(); // "Hello"
string trimStart = spaced.TrimStart(); // "Hello  "
string trimEnd = spaced.TrimEnd(); // "  Hello"
```

5. **字符串格式化**：
```csharp
// 使用 String.Format 格式化
string name = "John";
int age = 25;
string formatted = string.Format("My name is {0} and I'm {1} years old.", name, age);

// 使用插值字符串格式化
string formatted = $"My name is {name} and I'm {age} years old.";
```

> [!caution] 建议：使用字符串插值代替 `String.Format`



6. **格式化数字和日期**：
```csharp
// 数字格式化
double value = 12345.6789;
string num1 = value.ToString("C"); // 货币格式 "$12,345.68"
string num2 = value.ToString("N2"); // 数字格式 "12,345.68"
string num3 = value.ToString("F4"); // 固定点 "12345.6789"

// 日期格式化
DateTime now = DateTime.Now;
string date1 = now.ToString("yyyy-MM-dd"); // "2024-01-15"
string date2 = now.ToString("dd/MM/yyyy HH:mm:ss"); // "15/01/2024 14:30:45"
```


7. **字符串编码**：
```csharp
using System.Text;

string text = "Hello, 世界!";

// 转换为字节数组
byte[] utf8Bytes = Encoding.UTF8.GetBytes(text);
byte[] asciiBytes = Encoding.ASCII.GetBytes(text); // 非ASCII字符会丢失

// 从字节数组转换回字符串
string decoded = Encoding.UTF8.GetString(utf8Bytes);
```


8. **原始字符串字面量**（C# 11+，.NET 7+）：
```csharp
// 多行字符串
string json = """
    {
        "name": "John",
        "age": 30,
        "city": "New York"
    }
    """;

// 包含引号
string message = """He said, "Hello, World!" """;

// 插值原始字符串
string name = "John";
string greeting = $"""Hello, "{name}"!""";
```


## `StringBuilder`

对于频繁修改的字符串操作，使用 `StringBuilder` 可以提高性能。

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


## 性能调优

1. 使用 `StringBuilder` 进行大量字符串拼接
2. 使用字符串插值代替 `String.Format`
3. 注意字符串比较的文化差异
4. 避免在循环中拼接字符串
5. 考虑使用 `Span<char>` 处理高性能场景