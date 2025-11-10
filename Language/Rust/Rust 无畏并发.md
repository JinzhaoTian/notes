
安全且高效的处理并发编程是 Rust 的另一个主要目标。

### 1. 线程

Rust 标准库只提供了 1:1 线程模型实现。但是可以通过`crate`包来做m:n的线程模型。

#### 1\). 调用线程

为了创建一个新线程，需要调用 thread::spawn 函数并传递一个闭包，并在其中包含希望在新线程运行的代码。

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

`thread::sleep` 调用强制线程停止执行一小段时间，这会允许其他不同的线程运行。

在这里子线程是打印不完的：主线程打印完之后，就直接结束程序了

#### 2\). 使用 join 等待所有线程结束:

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

可以通过将 `thread::spawn` 的返回值储存在变量中来修复新建线程部分没有执行或者完全没有执行的问题。`thread::spawn` 的返回值类型是 `JoinHandle`。`JoinHandle` 是一个拥有所有权的值，当对其调用 `join` 方法时，它会等待其线程结束。通过调用 handle 的 join 会阻塞**当前线程**直到 handle 所代表的线程结束。

### 2. 消息传递

Rust 中一个实现消息传递并发的主要工具是 通道（channel）。

```rust
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {   // 子线程发送
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();  // 主线程接收
    println!("Got: {}", received);
}
```

首先使用标准库中的包`mpsc`\( 多个生产者，单个消费者（multiple producer, single consumer）的缩写\)，Rust 标准库实现通道的方式意味着一个通道可以有多个产生值的 发送（sending）端，但只能有一个消费这些值的 接收（receiving）端。

多个发送端可以通过`clone`来实现：

```rust
let tx1 = mpsc::Sender::clone(&tx);
```

`mpsc::channel` 函数返回一个元组：第一个元素是发送端，而第二个元素是接收端。

使用 `thread::spawn` 来创建一个新线程并使用 `move` 将 `tx` 移动到闭包中这样新建线程就拥有 `tx` 了。

通道的发送端有一个 send 方法用来获取需要放入通道的值。`send` 方法返回一个 `Result<T, E>` 类型，为了防止出错调用 `unwrap` 产生 `panic`。

通道的接收端有两个有用的方法：`recv` 和 `try_recv`。这里，我们使用了 `recv`，它是 receive 的缩写。这个方法会阻塞主线程执行直到从通道中接收一个值。一旦发送了一个值，`recv` 会在一个 `Result<T, E>` 中返回它。当通道发送端关闭，`recv` 会返回一个错误表明不会再有新的值到来了。

`try_recv` 不会阻塞，相反它立刻返回一个 `Result<T, E>`：Ok 值包含可用的信息，而 Err 值代表此时没有任何消息。

#### \*\* 通道与转移所有权

`send` 函数获取其参数的所有权并移动这个值归接收者所有。

### 3. 共享状态

Rust使用互斥器来进行共享内存操作，相当于《操作系统》里说过的PV操作。Rust里面有`Mutex<T>`API调用锁

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);   // 首先给内存上锁

    {
        let mut num = m.lock().unwrap();  // 获得锁
        *num = 6;
    }                                     // 离开域自动上锁

    println!("m = {:?}", m);
}
```

