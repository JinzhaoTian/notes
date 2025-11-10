## 最简单的程序：Hello World

```rust
fn main() {
    println!("Hello World!");
}
```

这部分与C语言类似，必须从main函数开始，fn相当于Python的def，用来标记一个函数，区别是不需要头文件。println不是一个函数，而是一个宏规则，所以函数名后面要加！。常见的输出方式有print!\(\)，println!\(\)。

## 语法

### 1. 变量与可变性

```rust
fn main(){
    let a = 123;     // 声明一个不可变变量，类型自动判断，这点和Python类似
    a = 234;         // 此语法错误，因为a不可变
    let a = a + 1;   // 语法正确，称之为重影(Shadowing)，也就是说重新绑定

    let mut a = 123; // 声明一个可变变量
    a = 234;         // 语法正确
    a = "abc";       // 语法错误，因为a已经被认定为了整型数据，不能被赋予字符串
    a++;             // 语法错误，Rust无++，--

    let a: u64 = 123;   // Rust也可以指定数据类型，只需要在变量名后面指定数据类型的名字

    const a: i32 = 123; // 注意：不可变变量不是常量，不可变变量可以重影，常量不行，常量后面的数据类型必须标注
}
```

注意：已声明的变量不用的话，编译会出现warning的。

### 2. 数据类型

#### 1\). 标量\(Scalar\)

* **整型**：有符号整型：`i8`, `i16`, `i32`, `i64`, `i128`，默认为`i32`。无符号整型：`u8`, `u16`, `u32`, `u64`, `u128`。此外还有`arch`的`isize`和`usize`，他们的所用字节数是根据你们电脑系统来定的，64位机器就是64位，32位机器就是32位。
* **浮点型**：32位浮点数：`f32`，64位浮点数：`f64`，默认`f64`。
* **布尔型**：布尔型用 `bool` 表示，值只能为 `true` 或 `false`。
* **字符型**：字符型用 `char` 表示，`char` 类型大小为 4 个字节，代表 Unicode标量值。

**2\). 组合类型\(Compound\)**

* **元组**：用一对`()`包括的一组数据，可以包含不同种类的数据

```rust
fn main(){
  let tup: (i32, f64, u8) = (500, 6.4, 1);
  let tup2 = (500, 6.4, "hello");
  let (x, y, z) = (1, 2, 3);      // 解构let

  let x = tup.0;
  let y = tup.1;
  let z = tup.2;

  println!("x is {}", x);
}
```

* **数组**：用一对`[]`包括的同类型数据

```rust
fn main(){
  let a = [1, 2, 3, 4, 5];
  let b: [i32;5] = [1, 2, 3, 4, 5];
  let c = [0; 20];             // c的每个元素都初始化为0

  println!("a[0] is {}", a[0]);

  println!("b has {} elements", b.len());
}
```

### 3. 函数

```rust
fn main() {
    println!("Hello, world!");
    println!("res is {}", function(2, 3));
}

fn function(x: i32, y: i32) -> i32 {
    return x + y;
}
```

function\(\)的声明放在main\(\)函数的前面或者后面都行, 参数一定要标明数据类型，若有返回值就要添加`->`并在其后标明数据类型，而且在函数结尾要有`return`，没有就不需要`->`和`return`。

```rust
fn main() {
    fn five() -> i32 {
        5   
    }
    println!("five() 的值为: {}", five());
}
```

函数还可以嵌套。函数显式返回用`return`，也可以隐式返回最后一个表达式\(不加分号的那种\)的值。

此外，Rust 中可以在一个用`{}`包括的块里编写一个较为复杂的表达式，这种表达式块叫做函数体表达式。Rust语句不返回值，C等中的语句会返回值。

```rust
fn main() {
    let x = 5;

    let y = {
        let x = 3;
        x + 1       // 最后一个表达式无分号，如果有分号的话就会变成一个语句，不返回值了
    };              // 表达式的结尾有分号

    println!("x 的值为 : {}", x);
    println!("y 的值为 : {}", y);
}
```

函数指针：

```rust
let x: fn(i32) -> i32 = foo;
```

### 4. 控制流

#### 1\). if语句

```rust
fn main() {
    let a = 12;
    let b;
    if a > 0 {
        b = 1;
    }  
    else if a < 0 {
        b = -1;
    }  
    else {
        b = 0;
    };
    println!("b is {}", b);
}
```

注意：Rust 中的 if 不存在单语句不用加 `{}` 的规则，不允许使用一个语句代替一个块。条件表达式必须是 `bool` 类型。

```rust
fn main() {
    let a = 3;
    let number = if a > 0 { 1 } else { -1 };  // 高级用法
    println!("number is {}", number);
}
```

注意：两个花括号中的类型必须一样！且必须有一个 else 及其后的表达式块，要整型都整型，要函数都函数。

#### 3\). while循环: 与C语言差不多

```rust
fn main() {
    let mut number = 1;
    while number != 4 {
        println!("{}", number);
        number += 1;
    }
    println!("end while");
}
```

#### 4\). for循环：在 C 语言中 for 循环使用三元语句控制循环，但是 Rust 中**没有**这种用法，for语句用于遍历一个迭代器。

```rust
fn main() {
    let mut a = [10, 20, 30, 40, 50];
    for i in a.iter() {             // 创造一个不可修改值的循环器
        println!("value is {}", i);
    }

    for i in a.iter_mut() {         // 创造一个可修改值的循环器
        println!("value is {}", i);
    }

    for i in 0..10 {                 // 左闭右开
        println!("value is {}", i);
    }

    for i in 0...10 {                // 左闭右闭
        println!("value is {}", i);
    }

    for (index, value) in (5..10).enumerate() {    // 记录循环了多少次了
        println!("index = {} and value = {}", index, value);
    }
}
```

循环标签:

```rust
'outer: for x in 0..10 {
    'inner: for y in 0..10 {
        if x % 2 == 0 { continue 'outer; } // Continues the loop over `x`.
        if y % 2 == 0 { continue 'inner; } // Continues the loop over `y`.
        println!("x: {}, y: {}", x, y);
    }
}
```

#### 5\). loop循环：Rust 语言有原生的无限循环结构

```rust
fn main() {
    let s = ['A', 'B', 'C', 'D', 'E', 'F'];
    let mut i = 0;
    loop {
        let ch = s[i];
        if ch == 'D' {
            break;
        }
        println!("No.{} is {}", i, ch);
        i += 1;
    }
}
```

