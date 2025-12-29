
## `Span<T>`

`Span<T>` 是 C# 7.2（.NET Framework 4.7.1+）引入的一个值类型，它提供了一种内存安全、高性能的方式来访问连续的内存区域。

### 核心特性

1. **高性能**：
	- **零分配（Zero-allocation）**：通常不分配堆内存
	- **栈上分配**：`Span<T>` 本身是值类型，存储在栈上
	- **避免复制**：直接引用现有内存，不创建副本

2. **内存安全**：
	- **边界检查**: 自动进行数组边界检查
	- **类型安全**: 编译器和运行时确保类型安全
	- **防止内存泄漏**: 不持有对象引用计数

3. **统一 API**
	- 可以处理数组、字符串、堆栈分配内存、非托管内存等

### 基本使用

1. **从字符串创建 `Span<char>`**：
```csharp
string text = "Hello, World!";
ReadOnlySpan<char> span = text.AsSpan(); // 只读Span

// 访问字符
char firstChar = span[0]; // 'H'
int length = span.Length; // 13

// 遍历
foreach (char c in span)
{
    Console.Write(c);
}
```

2. **切片操作（Slicing）**：
```csharp
string text = "Hello, World!";
ReadOnlySpan<char> span = text.AsSpan();

// 获取子字符串（零分配）
ReadOnlySpan<char> hello = span.Slice(0, 5);    // "Hello"
ReadOnlySpan<char> world = span.Slice(7, 5);    // "World"
ReadOnlySpan<char> rest = span.Slice(7);        // "World!"
```

### 应用场景

1. **高性能字符串处理**：
```csharp
// 统计字符串中特定字符的数量
public static int CountOccurrences(ReadOnlySpan<char> text, char target)
{
    int count = 0;
    for (int i = 0; i < text.Length; i++)
    {
        if (text[i] == target) count++;
    }
    return count;
}

// 使用
string input = "Hello, World!";
int count = CountOccurrences(input.AsSpan(), 'o'); // 2
```

2. **解析字符串**：
```csharp
// 高性能解析逗号分隔值
public static List<string> ParseCSV(ReadOnlySpan<char> input)
{
    var result = new List<string>();
    int start = 0;
    
    for (int i = 0; i < input.Length; i++)
    {
        if (input[i] == ',')
        {
            result.Add(input.Slice(start, i - start).ToString());
            start = i + 1;
        }
    }
    
    // 添加最后一个
    if (start < input.Length)
    {
        result.Add(input.Slice(start).ToString());
    }
    
    return result;
}
```

3. **字符串验证**：
```csharp
// 检查字符串是否为有效的数字
public static bool IsValidNumber(ReadOnlySpan<char> input)
{
    if (input.Length == 0) return false;
    
    int startIndex = 0;
    if (input[0] == '-') startIndex = 1;
    
    bool hasDigit = false;
    for (int i = startIndex; i < input.Length; i++)
    {
        if (!char.IsDigit(input[i])) return false;
        hasDigit = true;
    }
    
    return hasDigit;
}
```