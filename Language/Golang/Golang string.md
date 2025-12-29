Go 的 `string` 是一个不可变的字节序列，使用 UTF-8 编码来存储 Unicode 文本，但也可以存储任意二进制数据。

字符串的内部表示简单高效，通过指针和长度来引用底层的字节数组，底层使用 `stringStruct` 表示：
```go
// 运行时内部表示（简化版）
type stringStruct struct {
    str unsafe.Pointer  // 指向底层字节数组的指针
    len int             // 字符串的长度（字节数）
}
```

内存布局：
```
+--------+--------+-------------------+
| 指针   | 长度   | 底层字节数组      |
+--------+--------+-------------------+
| 8字节  | 8字节  | len个字节         |
+--------+--------+-------------------+
```

基本示例：
```go
package main

import (
    "fmt"
    "strings"
    "unicode/utf8"
)

func main() {
    // 创建和基本操作
    s := "Hello, 世界"
    fmt.Printf("原始字符串: %s\n", s)
    fmt.Printf("字节长度: %d\n", len(s))
    fmt.Printf("字符长度: %d\n", utf8.RuneCountInString(s))
    
    // 高效拼接
    var builder strings.Builder
    builder.Grow(50)
    builder.WriteString(s)
    builder.WriteString(" 👋")
    result := builder.String()
    fmt.Printf("拼接结果: %s\n", result)
    
    // 遍历字符
    fmt.Println("字符遍历:")
    for i, r := range s {
        fmt.Printf("位置 %d: %q (U+%04X)\n", i, r, r)
    }
}
```



## 核心特性

1. **不可变性**（重要特性）：
```go
s := "hello"
// 下面这行代码会导致编译错误：
// s[0] = 'H'  // 错误：字符串不可变
```

2. **零值**和**空字符串**：
```go
var s1 string      // 零值：空字符串 ""
s2 := ""          // 空字符串
s3 := "hello"     // 非空字符串
```

## 使用

### 创建

1. **字符串字面量**
```go
// 双引号（支持转义字符）
s1 := "Hello\tWorld\n"
fmt.Println(s1)  // Hello    World（换行）

// 反引号（原始字符串，不转义）
s2 := `Hello\tWorld\n`
fmt.Println(s2)  // Hello\tWorld\n（字面显示）
```

2. **从字节切片转换**
```go
// []byte 转 string（会复制数据）
bytes := []byte{72, 101, 108, 108, 111}
s := string(bytes)  // "Hello"

// string 转 []byte（会复制数据）
str := "Hello"
b := []byte(str)  // []byte{72, 101, 108, 108, 111}
```

3. **从符文切片转换**
```go
// []rune 转 string
runes := []rune{'你', '好', '世', '界'}
s := string(runes)  // "你好世界"

// string 转 []rune
str := "你好世界"
r := []rune(str)  // []rune{20320, 22909, 19990, 30028}
```


### 计算长度

1. **字节长度 vs 字符长度**
```go
s := "Hello, 世界"

// len() 返回字节数
fmt.Println(len(s))  // 13

// utf8.RuneCountInString() 返回字符数
fmt.Println(utf8.RuneCountInString(s))  // 9

// 遍历字符
for i, r := range s {
    fmt.Printf("字符%q 起始位置：%d\n", r, i)
}
```


### 编码

1. **UTF-8 编码**：
```go
s := "Hello, 世界"

// 遍历每个字节
for i := 0; i < len(s); i++ {
    fmt.Printf("%x ", s[i])
}
// 输出：48 65 6c 6c 6f 2c 20 e4 b8 96 e7 95 8c

// 遍历每个符文（Unicode码点）
for _, r := range s {
    fmt.Printf("%U ", r)
}
// 输出：U+0048 U+0065 U+006C U+006C U+006F U+002C U+0020 U+4E16 U+754C
```

### 操作

1. **拼接**
```go
// 方式1：使用 +（每次拼接都会创建新字符串）
s1 := "Hello"
s2 := "World"
s3 := s1 + ", " + s2

// 方式2：使用 fmt.Sprintf
s4 := fmt.Sprintf("%s, %s", s1, s2)

// 方式3：使用 strings.Builder（高效）
var builder strings.Builder
builder.WriteString("Hello")
builder.WriteString(", ")
builder.WriteString("World")
s5 := builder.String()

// 方式4：使用 strings.Join
s6 := strings.Join([]string{"Hello", "World"}, ", ")
```

2. **比较**
```go
s1 := "abc"
s2 := "abd"

fmt.Println(s1 == s2)  // false
fmt.Println(s1 < s2)   // true（按字典序比较）
fmt.Println(s1 > s2)   // false
```

3. **切片**
```go
s := "Hello, 世界"

// 按字节切片
sub1 := s[0:5]      // "Hello"
sub2 := s[7:]       // "世界"（注意：中文占用3字节）

// 转换为 []rune 后再切片
runes := []rune(s)
sub3 := string(runes[7:9])  // "世界"
```

4. **`strings` 包常用函数**
```go
import "strings"

s := "  Hello, World!  "

// 修剪
trimmed := strings.TrimSpace(s)     // "Hello, World!"

// 分割
parts := strings.Split("a,b,c", ",")  // []string{"a", "b", "c"}

// 替换
replaced := strings.Replace("foo foo", "foo", "bar", 1)  // "bar foo"

// 查找
contains := strings.Contains("hello world", "world")  // true
index := strings.Index("hello world", "world")       // 6
```

5. **`strconv` 包转换**
```go
import "strconv"

// 字符串与数值转换
num, _ := strconv.Atoi("123")      // string -> int
str := strconv.Itoa(123)           // int -> string

f, _ := strconv.ParseFloat("3.14", 64)  // string -> float64
s := strconv.FormatFloat(3.14, 'f', 2, 64)  // float64 -> string
```

