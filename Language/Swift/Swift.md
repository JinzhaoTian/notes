Swift 是 Apple 公司开发的一种编程语言，专门用来为 iPhone、iPad、Mac、Apple Watch 和 Apple TV 等设备编写 App。

## 关键特点

1. **Apple 官方主推的语言**：2014 年之前，Apple 开发主要用 Objective-C（一种更老、更复杂的语言）。2014 年 Apple 推出了 Swift，目标是让它**更简单、更安全、更好学**。现在 Apple 所有的官方教程、新框架（比如 SwiftUI）都是围绕 Swift 设计的。
2. **现代，对新手友好**：相比很多老语言，Swift 的语法更接近自然英语，读起来比较直观。比如你想打印一句话，只需要写：
```swift
print("Hello, world!")
```
你想定义一个变量存名字，写：
```swift
var name = "小明"
```

看起来就像在描述一件事，不需要记一堆奇怪的符号。
3. **安全**：Swift 在设计上会帮你避免很多常见错误。比如它要求你明确一个变量里到底有没有值，这样就不容易因为“空值”导致 App 崩溃。这对新手来说其实是好事——虽然一开始会觉得有点严格，但能帮你养成好习惯。
4. **性能好**：Swift 运行速度很快，接近 C 语言这类底层语言的性能，但写起来却比 C 简单得多。所以它既能做界面，也能处理复杂的计算。


## 语法

### 1. 变量与常量

Swift 使用 `var` 声明变量，`let` 声明常量。常量一旦赋值就不能修改，这有助于编写更安全的代码。

```swift
var name = "Alice"      // 变量，类型自动推断为 String
let pi = 3.14159        // 常量，类型自动推断为 Double

var age: Int = 25       // 显式指定类型
let greeting: String = "Hello"
```

Swift 是强类型语言，但支持类型推断，所以很多时候不需要写类型。

### 2. 数据类型

常见类型包括：

- `Int`、`Double`、`Float`：整数和浮点数
- `String`：字符串
- `Bool`：布尔值
- `Array`、`Dictionary`、`Set`：集合类型
- `Optional`：可选类型，表示值可能为 `nil`

```swift
let score: Int = 100
let price: Double = 19.99
let isActive: Bool = true
let names: [String] = ["Tom", "Jerry"]
let scores: [String: Int] = ["Tom": 90, "Jerry": 85]
```

### 3. 可选类型（Optional）

这是 Swift 非常有特色的语法。可选类型表示一个变量可能有值，也可能没有值（`nil`）。

```swift
var optionalName: String? = "Alice"
optionalName = nil

// 可选绑定
if let name = optionalName {
    print("名字是 \(name)")
} else {
    print("没有名字")
}

// 强制解包（不推荐，除非确定有值）
let forcedName = optionalName!
```

还有 `guard let` 用于提前退出：

```swift
func greet(_ name: String?) {
    guard let name = name else {
        print("名字为空")
        return
    }
    print("你好，\(name)")
}
```

### 4. 控制流

#### if 语句

```swift
let score = 85
if score >= 90 {
    print("优秀")
} else if score >= 60 {
    print("及格")
} else {
    print("不及格")
}
```

#### switch 语句

Swift 的 `switch` 不需要写 `break`，而且必须覆盖所有情况。

```swift
let grade = "A"
switch grade {
case "A":
    print("优秀")
case "B":
    print("良好")
default:
    print("其他")
}
```

#### 循环

```swift
for i in 1...5 {
    print(i)
}

var count = 0
while count < 3 {
    print(count)
    count += 1
}
```

### 5. 函数

函数使用 `func` 关键字定义。参数可以有外部参数名和内部参数名。

```swift
func add(a: Int, b: Int) -> Int {
    return a + b
}

let result = add(a: 3, b: 5)

// 使用外部参数名
func greet(person name: String) -> String {
    return "Hello, \(name)"
}
greet(person: "Alice")
```

### 6. 闭包（Closure）

闭包是自包含的代码块，类似于其他语言中的匿名函数或 lambda。

```swift
let multiply = { (a: Int, b: Int) -> Int in
    return a * b
}

let numbers = [3, 1, 4, 1, 5]
let sorted = numbers.sorted { $0 < $1 }
```

### 7. 结构体与类

Swift 中结构体（`struct`）是值类型，类（`class`）是引用类型。

```swift
struct Point {
    var x: Int
    var y: Int
}

class Person {
    var name: String
    init(name: String) {
        self.name = name
    }
    func sayHello() {
        print("Hello, I'm \(name)")
    }
}
```

### 8. 枚举

Swift 的枚举非常强大，可以有关联值。

```swift
enum Direction {
    case north, south, east, west
}

enum Result {
    case success(String)
    case failure(Error)
}
```

### 9. 协议与扩展

协议定义一组要求，类、结构体或枚举都可以遵循。

```swift
protocol Greetable {
    func greet() -> String
}

struct Student: Greetable {
    func greet() -> String {
        return "I'm a student"
    }
}

extension String {
    func reversedText() -> String {
        return String(self.reversed())
    }
}
```

### 10. 错误处理

Swift 使用 `do-catch` 进行错误处理。

```swift
enum FileError: Error {
    case notFound
}

func readFile() throws {
    throw FileError.notFound
}

do {
    try readFile()
} catch {
    print("出错了：\(error)")
}
```
