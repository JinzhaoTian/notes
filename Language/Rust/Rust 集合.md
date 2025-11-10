
集合可以包含多个值。不同于内建的数组和元组类型，这些集合指向的数据是储存在堆上的，这意味着数据的数量不必在编译时就已知，并且还可以随着程序的运行增长或缩小。

### 1. vector

`Vec` 是一个由标准库提供的类型，它可以存放任何类型，而当 `Vec` 存放某个特定类型时，那个类型位于尖括号中。在更实际的代码中，一旦插入值 Rust 就可以推断出想要存放的类型，所以你很少会需要这些类型注解。更常见的做法是使用初始值来创建一个 `Vec`，而且为了方便 Rust 提供了 `vec!` 宏。

```rust
// 常见声明方式
let v: Vec<i32> = Vec::new();
let v = vec![1, 2, 3];

// 往后面添加元素
let mut v = Vec::new();
v.push(5);    // 根据添加的数据类型推断出vec的类型
v.push(6);
```

vector的访问方式有两种：

```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
println!("The third element is {}", third);

match v.get(2) {
    Some(third) => println!("The third element is {}", third),
    None => println!("There is no third element."),
}
```

这两个不同的获取第三个元素的方式分别为：使用`&` 和 `[]` 返回一个引用；或者使用 `get` 方法以索引作为参数来返回一个 `Option<&T>`。当读取的下标超出范围时，对于第一个`[]`方法，当引用一个不存在的元素时 Rust 会造成`panic`。这个方法更适合当程序认为尝试访问超过`vector`结尾的元素是一个严重错误的情况，这时应该使程序崩溃。当`get`方法被传递了一个数组外的索引时，它不会`panic`而是返回`None`。当偶尔出现超过`vector`范围的访问属于正常情况的时候可以考虑使用它。

遍历一个`vector`:

```rust
let v = vec![100, 32, 57];
for i in &v {
    println!("{}", i);
}

let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;     // * 号是解引用运算符
}
```

vector 只能储存相同类型的值。这是很不方便的；绝对会有需要储存一系列不同类型的值的用例。幸运的是，枚举的成员都被定义为相同的枚举类型，所以当需要在 vector 中储存不同类型值时，我们可以定义并使用一个枚举：

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];          // 初始化一个vector
```

### 2. 字符串

Rust 的核心语言中只有一种字符串类型：`str`，字符串 `slice`，它通常以被借用的形式出现，`&str`。前面讲到了 字符串 `slice`：它们是一些储存在别处的 UTF-8 编码字符串数据的引用。比如字符串字面值被储存在程序的二进制输出中，字符串 `slice` 也是如此。

称作 `String` 的类型是由标准库提供的，而没有写进核心语言部分，它是可增长的、可变的、有所有权的、UTF-8 编码的字符串类型。

#### 1\). 字符串的声明

```rust
let mut s = String::new();      // 声明一个空String
s.push_str("bar");    // 附加一个字符串
s.push('l');          // 附加一个字符


let s = String::from("initial contents");   // 声明方法二

let data = "initial contents";  // 声明一个str
let s = data.to_string();       // 转化成String
```

#### 2\). 字符串的拼接：

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // 注意 s1 被移动了，不能继续使用
```

上述字符串的拼接使用的是 `+` 运算符，相当于使用了 `add` 函数，`add`函数的形式像这样的：

```rust
fn add(self, s: &str) -> String {
```

这并不是标准库中实际的签名；标准库中的 `add` 使用泛型定义。这里我们看到的 `add` 的签名使用具体类型代替了泛型，这也正是当使用 `String` 值调用这个方法会发生的。**这个语句会获取 s1 的所有权，附加上从 s2 中拷贝的内容，并返回结果的所有权。**简而言之，这句话之后，s1失效了，但是s2没失效。

字符串的拼接还可以有下面的形式，使用 `format!` 宏：

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{}-{}-{}", s1, s2, s3);
```

#### 3\). 字符串的引用

字符串的引用不能用 `[]` ，使用如下代码会出错：

```rust
let s1 = String::from("hello");
let h = s1[0];     // 报错
```

这是因为 `String` 是一个 `Vec<u8>` 的封装，而 `UTF-8` 编码是一种针对Unicode的**可变长度**字符编码，也是一种前缀码。所以在引用的时候就会出现错误。

但是可以这么来做，进行访问字符：

```rust
for c in "नमस्ते".chars() {   // 返回字符形式的
    println!("{}", c);
}

for b in "नमस्ते".bytes() {   // 返回字节形式的数字
    println!("{}", b);
}
```

### 3. 哈希map

#### 1\). 新建哈希

必须首先 `use` 标准库中集合部分的 `HashMap`。由于哈希使用的相对少，所以标准库中对 `HashMap` 的支持也相对较少，例如，并没有内建的构建宏。哈希 map 将它们的数据储存在堆上，一个哈希里面的键值对类型一定要相同。

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

#### 2\). 所有权

对于像 `i32` 这样的实现了 `Copy` trait 的类型，其值可以拷贝进哈希 map。对于像 `String` 这样拥有所有权的值，其值将被移动而哈希 map 会成为这些值的所有者。

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// 这里 field_name 和 field_value 不再有效
```

#### 3\). 访问哈希

方法一：通过 `get()` :

```rust
let team_name = String::from("Blue");
let score = scores.get(&team_name);
```

注意： `get` 返回 `Option<V>`，所以结果被装进 `Some`；如果某个键在哈希 map 中没有对应的值，`get` 会返回 `None`。这时就要用某种第六章提到的方法之一来处理 `Option`。

方法二：使用 `for` 循环:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {    // 输出任意顺序
    println!("{}: {}", key, value);
}
```

#### 4\). 更新哈希map

```rust
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);  // 直接更新

scores.entry(String::from("Blue")).or_insert(50);  // 使用 entry 方法只在键没有对应一个值时插入，如果有值就不更新
```

哈希 map 有一个特有的 API，叫做 `entry`，它获取我们想要检查的键作为参数。`entry` 函数的返回值是一个枚举，`Entry`，它代表了可能存在也可能不存在的值。`Entry` 的 `or_insert` 方法在键对应的值存在时就返回这个值的 `Entry`，如果不存在则将参数作为新值插入并返回修改过的 `Entry`。
