
结构体与C++中的类似：

### 1. 结构体的声明和使用

```rust
struct User {  // 声明
    username: String,
    email: String,
    sign_in_count: u64;
    active: bool,
}

let user1 = User{   // 初始化一个实例
    email: String::from("someone@email.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};

let mut user2 = User{
    email: String::from("someone@email.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
user2.email = String::from("anotheremail@email.com");
```

如果声明了一个结构体实例user1，但是想声明另一个与user1大部分数据相同的结构体实例user2，就可以使用下面的方法：

```rust
let user2 = User{
    email:String::from("another@example.com"),
    username: String::from("anotherusername567"),
    ..user1    // 此处相当于除了上面特别声明的不一样之外，剩下的都与user1相同
};
```

### 2. 元组结构体

元组结构体的内部变量不需要名字，只有变量类型

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

### 3. 通过派生Trait增加实用功能

当我们尝试输出结构体时，用下面的方法往往会出现错误：

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!("rect1 is {}", rect1);
}
```

`println!`宏能处理很多类型的格式，不过，`{}` 默认告诉 `println!` 使用被称为 `Display` 的格式：意在提供给直接终端用户查看的输出。目前为止见过的基本类型都默认实现了 `Display`，因为它就是向用户展示其他任何基本类型的唯一方式。不过对于结构体，`println!` 应该用来输出的格式是不明确的，因为这有更多显示的可能性：是否需要逗号？需要打印出大括号吗？所有字段都应该显示吗？由于这种不确定性，Rust 不会尝试猜测我们的意图，所以结构体并没有提供一个 `Display` 实现。

但是我们如果在结构体定义之前加上 \#\[derive\(Debug\)\] 注解，然后使用一些特殊的格式输出\(这些均出现在错误信息中\)。

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!("rect1 is {:?}", rect1);      // 第一种格式
    println!("rect1 is {:#?}", rect1);     // 第二种格式
}
```

Rust 为我们提供了很多可以通过 `derive` 注解来使用的 `trait`，他们可以为我们的自定义类型增加实用的行为。

### 4. 方法语法

**方法**与函数类似：它们使用 `fn` 关键字和名称声明，可以拥有参数和返回值，同时包含在某处调用该方法时会执行的代码。不过方法与函数是不同的，因为它们在结构体的上下文中被定义（或者是枚举或 trait 对象的上下文），并且它们第一个参数总是 `self`，它代表调用该方法的结构体实例。

#### 1\). 定义方法

Rust 通过impl关键字提供了使用方法调用语法（method call syntax）。这样写的话有利于组织：我们将某个类型实例能做的所有事情都一起放入`impl`块中，而不是让将来的用户在我们的库中到处寻找`Rectangle`的功能。

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {  // impl 是 implementation 的缩写
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()    // 在实例上直接调用方法
    );
}
```

方法的第一参数比较特殊，`&self`。它有3种变体：`self`，`&self`和`&mut self`。方法可以选择获取`self`的所有权，或者像我们这里一样不可变地借用`self`，或者可变地借用`self`，就跟其他参数一样。这里选择`&self`的理由是：我们并不想获取所有权，只希望能够读取结构体中的数据，而不是写入。如果想要在方法中改变调用方法的实例，需要将第一个参数改为`&mut self`。

也可以把`impl`块写成多个:

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

还可以链式方法调用：

```rust
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radius * self.radius)
    }

    fn grow(&self, increment: f64) -> Circle {
        Circle { x: self.x, y: self.y, radius: self.radius + increment }
    }
}

fn main() {
    let c = Circle { x: 0.0, y: 0.0, radius: 2.0 };
    println!("{}", c.area());

    let d = c.grow(2.0).area();  // 链式
    println!("{}", d);
}
```

#### 2\). 关联函数

`impl`块的另一个有用的功能是：允许在`impl`块中定义 不以`self`作为参数的函数。这被称为关联函数（associated functions），因为它们与结构体相关联。它们仍是函数而不是方法，因为它们并不作用于一个结构体的实例。

```rust
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}
```

使用结构体名和`::`语法来调用这个关联函数：比如`let sq = Rectangle::square(3);` 。这个方法位于结构体的命名空间中：`::` 语法用于关联函数和模块创建的命名空间。这在前面我们已经用过了，比如使用 `String::from` 关联函数了

