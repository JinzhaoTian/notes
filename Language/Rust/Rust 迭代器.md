
Rust 的设计灵感来源于很多现存的语言和技术。其中一个显著的影响就是**函数式编程**（functional programming）。函数式编程风格通常包含将函数作为参数值或其他函数的返回值、将函数赋值给变量以供之后执行等等。

### 1. 闭包

闭包（Closure）也叫Lambda表达式或匿名函数。

不像普通函数，闭包可以对参数和返回类型进行推断，大多数时候都不需要写出来。以下定义都是合法的：

```rust
|| 42;
|x| x + 1;
|x:i32| x + 1;
|x:i32| -> i32 { x + 1 };
```

在上面的例子中，如果是单行语句且没有标注返回类型的时候，花括号是可选的。

闭包可以像任何其他对象一样绑定到某个变量：

```rust
let f = |x| x + 1;
```

然后可以像调用函数一样调用闭包：

```rust
f(10);
```

也可以在定义的地方直接调用：

```rust
let r = (|x| x + 1)(2); // r == 3
```

闭包可以捕获外部的环境变量（自由变量）。

闭包捕获变量的方式分为三类：引用（&T）、可变引用（&mut T）和值（T）。

捕获变量时，闭包会根据上面列出的顺序（从约束最少到约束最多），优先按引用捕获，必要时才会使用后面的捕获方式：

```rust
let x = 10;
// 闭包按引用捕获变量x，因为println!只需要引用参数
let show_x = || println!("x = {}", x);
show_x();
```

外部变量的引用保存在show\_x对象中，对外部变量的借用持续到show\_x离开作用域为止。

下面是一个捕获可变引用的例子：

```rust
let mut count = 0;
// 闭包按可变引用捕获变量count 
// incr也必须是可变的，因为它持有可变引用，调用incr会改变闭包的状态
let mut incr = || { count += 1; println!("count = {}", count); };
incr();
```

下面的代码演示了闭包转移捕获变量所有权时的情况（捕获变量的值）：

```rust
// b是不可复制类型，因此按值捕获时所有权会转移
let b = String::from("Hello World");
let f = move || {
    println!("b = {}", b);
};
f();

println!("b = {}", b);  // 再访问b，会报错
```

由于`String`的所有者只有一个，所以，在捕获变量的值的时候 `b` 的所有权被转移进了闭包内。如果 `b` 是 `i32` 类型的话，仍然可以访问。

闭包另外有一些高级用法，有兴趣可以了解一下书。

### 2. 迭代器

迭代器模式允许你对一个项的序列进行某些处理。迭代器（iterator）负责遍历序列中的每一项和决定序列何时结束的逻辑。当使用迭代器时，我们无需重新实现这些逻辑。

在 Rust 中，迭代器是 惰性的（lazy），这意味着在调用方法使用迭代器之前它都不会有效果。

```rust
let v1 = vec![1, 2, 3];

let v1_iter = v1.iter();

for val in v1_iter {
    println!("Got: {}", val);
}
```

#### 1\). `Iterator` trait 和 `next` 方法

迭代器都实现了一个叫做 Iterator 的定义于标准库的 trait。这个 trait 的定义看起来像这样：

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // 此处省略了方法的默认实现
}
```

注意这里有一下我们还未讲到的新语法：`type Item` 和 `Self::Item`，他们定义了 trait 的 关联类型（associated type）。第十九章会深入讲解关联类型，不过现在只需知道这段代码表明实现 `Iterator` trait 要求同时定义一个 `Item` 类型，这个 `Item` 类型被用作 `next` 方法的返回值类型。换句话说，`Item` 类型将是迭代器返回元素的类型。

`next` 是 `Iterator` 实现者被要求定义的唯一方法。`next` 一次返回迭代器中的一个项，封装在 `Some` 中，当迭代器结束时，它返回 `None`。

可以直接调用迭代器的 `next` 方法:

```rust
#[test]
fn iterator_demonstration() {
    let v1 = vec![1, 2, 3];

    let mut v1_iter = v1.iter();  // 此处v1_iter需要是mut的

    assert_eq!(v1_iter.next(), Some(&1));
    assert_eq!(v1_iter.next(), Some(&2));
    assert_eq!(v1_iter.next(), Some(&3));
    assert_eq!(v1_iter.next(), None);
}
```

注意 `v1_iter` 需要是可变的：在迭代器上调用 `next` 方法改变了迭代器中用来记录序列位置的状态。换句话说，代码 消费（consume）了，或使用了迭代器。每一个 `next` 调用都会从迭代器中消费一个项。使用 `for` 循环时无需使 `v1_iter` 可变因为 `for` 循环会获取 `v1_iter` 的所有权并在后台使 `v1_iter` 可变。

另外需要注意到从 `next` 调用中得到的值是vector的不可变引用。`iter` 方法生成一个不可变引用的迭代器。如果我们需要一个获取 `v1` 所有权并返回拥有所有权的迭代器，则可以调用 `into_iter` 而不是 `iter`。类似的，如果我们希望迭代可变引用，则可以调用 `iter_mut` 而不是 `iter`。

#### 2\). 消费迭代器的方法：

调用 next 方法的方法被称为 消费适配器，因为调用他们会消耗迭代器。比如`sum`方法，这个方法获取迭代器的所有权并反复调用 next 来遍历迭代器，因而会消费迭代器。当其遍历每一个项时，它将每一个项加总到一个总和并在迭代完成时返回总和：

```rust
fn iterator_sum() {
    let v1 = vec![1, 2, 3];

    let v1_iter = v1.iter();

    let total: i32 = v1_iter.sum();   // 6
}
```
