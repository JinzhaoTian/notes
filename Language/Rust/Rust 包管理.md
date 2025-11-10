
### Cargo

Cargo 有点像 Java 中的 Maven 与 Ant 的合体，是一个依赖管理工具，不过更全能一些，用来管理依赖，编译，运行等。

```rust
$ cargo new hello_cargo    // 创建项目
$ cd hello_cargo           // 切换到项目路径下
```

路径下一般会有一个 `src` 文件夹，一个 `Cargo.toml` 文件，以及版本控制 `.gitignore` 文件。

```bash
cargo build              // 编译并生成可执行文件
cargo check              // 编译，不生成可执行文件（快）
```

生成的可执行文件一般在`./target/debug/`下，用`cargo build`命令时由于会生成可执行文件，所以速度比`cargo check`慢一点。

```bash
cargo run                // build并执行项目文件
```


## 包\(Packages\)，包装箱\(Crates\)与模块\(Modules\)

### 1. 包和crate

crate 是一个二进制项或者库。crate root 是一个源文件，Rust 编译器以它为起始点，并构成你的 crate 的根模块。包（package） 是提供一系列功能的一个或者多个 crate。一个包会包含有一个 `Cargo.toml` 文件，阐述如何去构建这些 crate。

#### 1\). 包的构建规则

* 一个包中至多**只能**包含一个**库 crate**\(library crate\)；
* 包中可以包含任意多个**二进制 crate**\(binary crate\)；
* 包中至少包含一个 crate，无论是库的还是二进制的。  

可以使用`cargo new`构建包

### 2. 模块

通过执行 `cargo new --lib restaurant`，来创建一个新的名为 `restaurant` 的库。然后将示例中所罗列出来的代码放入 `src/lib.rs` 中，来定义一些模块和函数。

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn server_order() {}

        fn take_payment() {}
    }
}
```

我们定义一个模块，是以 `mod` 关键字为起始，然后指定模块的名字（本例中叫做 front\_of\_house），并且用花括号包围模块的主体。在模块内，我们还可以定义其他的模块，就像本例中的 `hosting` 和 `serving` 模块。模块还可以保存一些定义的其他项，比如结构体、枚举、常量、特性、或者函数。

写模块的时候也要注意公有还有私有性，Rust 中默认所有项（函数、方法、结构体、枚举、模块和常量）都是私有的。父模块中的项不能使用子模块中的私有项，但是子模块中的项可以使用他们父模块中的项。可以使用`pub`关键字变成公有。

### 3. 路径用于引用模块树中的项

来看一下 Rust 如何在模块树中找到一个项的位置，我们使用路径的方式，就像在文件系统使用路径一样。如果我们想要调用一个函数，我们需要知道它的路径。

路径有两种形式：

* 绝对路径（absolute path）：从 `crate` 根开始，以 `crate` 名或者字面值 `crate` 开头。
* 相对路径（relative path）：从当前模块开始，以 `self`、`super` 或当前模块的标识符开头。

绝对路径和相对路径都后跟一个或多个由双冒号（`::`）分割的标识符。

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

### 4. 使用 `use` 关键字将名称引入作用域

我们可以一次性将路径引入作用域，然后使用 use 关键字调用该路径中的项，就如同它们是本地项一样。

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```

还可以使用`as`关键字提供新的名称

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
}

fn function2() -> IoResult<()> {
    // --snip--
}
```
