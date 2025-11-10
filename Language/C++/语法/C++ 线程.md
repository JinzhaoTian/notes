C++ 11 中引入了一个功能完整的线程库，头文件主要为 `<thread>`。在 C++ 11之前，C++ 标准库没有原生的线程支持，开发者只能使用平台特定的 API，导致代码可移植性差。

## 核心组件

1. **`std::thread`**：线程类，通过创建一个 `std::thread` 对象，并传入一个可调用对象（函数、Lambda 表达式、函数对象等），即可启动一个新线程。
```cpp
#include <iostream>
#include <thread>

void helloFunction() {
    std::cout << "Hello from thread!" << std::endl;
}

int main() {
    // 创建并启动线程，执行helloFunction
    std::thread t(helloFunction);
    
    // 等待线程t执行完毕
    t.join(); 
    
    return 0;
}
```

2. **`std::mutex`**：基本的互斥锁，用于保护共享数据。
```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <mutex>

int shared_counter = 0;
std::mutex mtx; // 创建一个全局的 mutex 对象

void increment() {
    for (int i = 0; i < 100000; ++i) {
        mtx.lock();   // 加锁：获取钥匙
        shared_counter++; // 临界区代码
        mtx.unlock(); // 解锁：归还钥匙
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);

    t1.join();
    t2.join();

    // 现在结果永远是 200000
    std::cout << "Final counter value: " << shared_counter << std::endl;
    return 0;
}
```

> [!warning] 不推荐手动加锁
> 手动调用 `lock()` 和 `unlock()` 非常危险，因为如果在 `lock()` 和 `unlock()` 之间发生了异常，或者忘记了调用 `unlock()`，就会导致锁永远无法释放，其他所有线程都会无限期等待，这被称为死锁。

3. **`std::lock_guard`**：RAII 风格的锁管理器，**在构造时自动获取锁，在析构时（比如离开作用域时）自动释放锁**，防止忘记解锁导致的死锁。
	- C++ 14 为互斥量增加了 `std::lock_guard` 的模板推导，使得代码可以更简洁。
```cpp
#include <thread>
#include <mutex>

std::mutex g_mutex;
int g_sharedData = 0;

void safeIncrement() {
    std::lock_guard<std::mutex> lock(g_mutex);
    for (int i = 0; i < 100000; ++i) {
        ++g_sharedData;
    }
}
```

4. **`std::unique_lock`**：RAII 风格的锁管理器，在构造时自动获取锁，在析构时自动释放锁。
	- 允许手动控制加锁和解锁的时机（比 `std::lock_guard` 更灵活）。
```cpp
{
    // 构造时自动加锁，类似 std::lock_guard
    std::unique_lock<std::mutex> lock(mtx); 
    shared_data++; // 安全地访问共享数据
    // 析构时自动解锁
}

{
    // 使用 std::defer_lock 参数，构造时不加锁
    std::unique_lock<std::mutex> lock(mtx, std::defer_lock);
    
    // ... 执行一些不需要锁保护的操作 ...
    
    lock.lock();   // 手动加锁
    shared_data++; // 临界区操作
    lock.unlock(); // 手动解锁（可选）
    
    // ... 更多操作 ...
    // 如果锁仍被持有，析构时会自动解锁
}

{
	// 使用 std::try_to_lock 参数，尝试加锁而不阻塞
    std::unique_lock<std::mutex> lock(mtx, std::try_to_lock); 
    
    if (lock.owns_lock()) { // 检查是否成功获取锁
        // 成功获取锁，执行临界区操作
        shared_data++;
        std::cout << "Lock acquired successfully!" << std::endl;
    } else {
        // 未能获取锁，执行替代操作
        std::cout << "Could not acquire lock, doing something else..." << std::endl;
    }
}

{
	// 设置加锁的超时时间
    std::unique_lock<std::mutex> lock(mtx, std::chrono::milliseconds(100));
    
    if (lock.owns_lock()) {
        // 在100毫秒内成功获取锁
        shared_data++;
        std::cout << "Lock acquired within timeout!" << std::endl;
    } else {
        // 超时未能获取锁
        std::cout << "Timeout: could not acquire lock" << std::endl;
    }
}
```

> [!tip]- 使用 `std::unique_lock` 可以精确控制锁的持有范围
> ```cpp
> void fine_grained_locking() {
>     std::unique_lock<std::mutex> lock(mtx);
>   
>     // 执行需要锁保护的操作
>     shared_data++;
>    
>     // 临时释放锁，让其他线程有机会执行
>     lock.unlock();
>    
>     // 执行一些耗时但不需要锁保护的操作
>     std::this_thread::sleep_for(std::chrono::milliseconds(100));
>    
>     // 重新获取锁
>     lock.lock();
>    
>     // 继续执行需要锁保护的操作
>     shared_data--;
> }
> ```

5. **`std::condition_variable`**：条件变量是一种同步原语，用于在多线程程序中实现线程间的等待和通知机制。允许线程在某个条件不满足时**主动等待**，并在条件可能满足时被**通知唤醒**。
	- 线程不是通过忙等待（busy-waiting）来检查条件，而是**休眠等待**
	- 当其他线程改变了共享状态并使得条件可能满足时，**通知**等待的线程
	- 被通知的线程**醒来检查条件**，如果条件满足则继续执行，否则继续等待
	- **核心成员函数**：
		- `wait(lock, predicate)`：等待直到条件满足（推荐用法）
		- `wait(lock)`：等待通知（可能虚假唤醒）
		- `notify_one()`：唤醒一个等待线程
		- `notify_all()`：唤醒所有等待线程
```cpp
std::queue<int> queue;
std::mutex mtx;
std::condition_variable cv;

void consumer_good() {
    while (true) {
        std::unique_lock<std::mutex> lock(mtx);
        
        // 等待条件满足：队列不为空
        cv.wait(lock, []{ return !queue.empty(); });
        
        int value = queue.front();
        queue.pop();
        lock.unlock();
        
        // 处理数据...
    }
}
```

> [!info]- 为什么需要 `std::condition_variable` ？
> 当没有条件变量 `std::condition_variable` 时，线程会进入忙等待状态：
> ```cpp
> std::queue<int> queue;
> std::mutex mtx;
>
> // 消费者线程 - 错误的忙等待方式
> void consumer_bad() {
>     while (true) {
>        std::lock_guard<std::mutex> lock(mtx);
>        if (!queue.empty()) {
>            int value = queue.front();
>            queue.pop();
>            // 处理数据...
>        }
>        // 如果没有数据，就不断循环检查，浪费CPU
>     }
> }
> ```
> 这种忙等待方式会大量消耗 CPU 资源。

> [!hint] `std::condition_variable` 必须与 `std::unique_lock` 配合使用
> 条件变量必须与 `std::unique_lock` 一起使用，不能与 `std::lock_guard` 一起使用。

6. **`std::future`** / **`std::promise`**： 用于在线程间传递异步操作的结果。
	- `std::promise`：输入端
	- `std::future`：输出端，用于获取结果。

7. **`std::async`**： 一个高级接口，可以方便地启动一个异步任务并返回一个 `std::future` 来获取结果。

8. （C++ 14）**`std::shared_timed_mutex`**：读写锁（共享-定时互斥量），允许多个读线程同时访问，但写线程独占。

9. （C++ 17）**`std::shared_mutex`**：读写锁，允许多个线程同时进行读操作，但写操作必须是独占的。是 C++ 14 的 `std::shared_timed_mutex` 的非定时版本，性能可能更好。

10. （C++ 17）**`std::scoped_lock`**： 用于同时锁定多个互斥量而不会死锁，是 `std::lock_guard` 的多互斥量版本，推荐替代。

11. （C++ 17）**并行 STL 算法**： 许多 STL 算法（如 `std::sort`, `std::for_each`）现在可以指定执行策略（`std::execution::par`）来并行执行，这背后就使用了线程池。

> [!info] C++ 20 重大飞跃与现代化
> C++ 20 对并发编程的支持进行了大规模增强，引入了更现代、更易用的工具。

12. （C++ 20）**`std::jthread`**：智能线程，对 `std::thread` 的重大改进，`j` 代表 `joining` 或 `stop token`。
	- **自动联结（Automatic Joining）**：在 `std::jthread` 的析构函数中，如果线程仍可联结（joinable），它会自动调用 `join()`，这彻底避免了因忘记 `join` 或 `detach` 而导致程序崩溃的问题。
	- **协作式中断（Cooperative Interruption）**：内部集成了 `std::stop_source` 和 `std::stop_token` 机制，允许外部请求线程停止，线程内部可以定期检查并优雅退出。

```
#include <iostream>
#include <thread>

void task(std::stop_token stoken) {
    while (!stoken.stop_requested()) {
        std::cout << "Working...\n";
        std::this_thread::sleep_for(1s);
    }
    std::cout << "Thread stopped gracefully.\n";
}

int main() {
    std::jthread t(task); // 创建并启动线程
    std::this_thread::sleep_for(3s);
    // main函数结束时，jthread t的析构函数会：
    // 1. 请求停止 (t.request_stop())
    // 2. 自动等待线程结束 (t.join())
    return 0;
}
```

13. （C++ 20）**`std::atomic<std::shared_ptr>`** / **`std::atomic<std::weak_ptr>`**：原子智能指针，对智能指针的读写操作变成原子的，无需额外的锁，简化了并发环境下的内存管理。

14. （C++ 20）**`std::counting_semaphore`** 和 **`std::binary_semaphore`**：信号量是一种更通用的同步原语，用于控制对共享资源的访问数量，可以用来实现生产者-消费者模型等。

15. （C++ 20）**`std::latch`** / **`std::barrier`**：闩（Latch）与屏障（Barrier）
	- `std::latch`： 一种一次性使用的线程同步点，允许线程阻塞直到计数器减为零。不可复用。  
    - `std::barrier`： 一种可复用的线程同步机制，允许一组线程在多个阶段同步。当所有线程都到达屏障点后，它们才会继续执行，并且屏障会被重置以供下一个阶段使用。

16. （C++ 20）**协程支持**
    - 虽然协程本身不是线程（它是用户态的、更轻量级的并发体），但它是 C++ 20 异步编程模型的核心组成部分，与线程库紧密协作，为未来的高性能并发库奠定了基础。


## 基本用法

### 创建线程

创建一个 `std::thread` 对象，需要传入一个可调用对象（函数、Lambda 表达式、函数对象等）

1. 可调用对象为**普通函数**：
```cpp
#include <iostream>
#include <thread>

void hello() {
    std::cout << "Hello, World!" << std::endl;
}

int main() {
    std::thread t(hello);
    t.join(); // 等待线程结束
    return 0;
}
```

2. 可调用对象为**带参数的函数**：函数参数分为是**按值传递**、**按引用传递**和**按指针传递**
	- （默认）按值传递参数
	- 按引用传递需要使用 `std::ref` 或`std::cref` 
```cpp
#include <iostream>
#include <thread>

void greet(const std::string& name) {
    std::cout << "Hello, " << name << "!" << std::endl;
}

void update(int& value) {
    value = 100;
}

void processArray(int* arr, size_t size) {
    for (size_t i = 0; i < size; ++i) {
        arr[i] *= 2;
    }
}

int main() {
    std::thread t(greet, "Alice");                 // 按值传递
    t.join();
    
    int local_value = 0;
    std::thread t(update, std::ref(local_value));  // 按引用传递，使用 std::ref 来传递引用
    t.join();
    std::cout << "local_value after update: " << local_value << std::endl; // 输出100
    
    int data[] = {1, 2, 3, 4, 5};
    size_t size = sizeof(data) / sizeof(data[0]);
    
    std::thread t(processArray, data, size);       // 按指针传递
    t.join();
    
    return 0;
}
```

> [!info]- 创建按引用传递参数函数线程使用 `std::ref` 或 `std::cref` 的主要原因
> 1. `std::thread` 的构造函数**默认拷贝**所有参数到线程的内部存储中。
> 2. `std::thread` 使用**模板参数推导**和**完美转发**，当它看到 `std::ref` 包装的参数时，会保留引用语义而不是进行拷贝。
> 	- `std::ref`（非 const 引用）或 `std::cref`（const 引用）会创建一个引用包装器，告诉线程不要拷贝参数，而是传递引用。

3. 可调用对象为**Lambda 表达式**：
```cpp
#include <iostream>
#include <thread>

int main() {
    std::thread t([]() {
        std::cout << "Hello from inline lambda!" << std::endl;
    });
    t.join();
    return 0;
}
```

4. 可调用对象为**函数对象（Functor）**：
```cpp
#include <iostream>
#include <thread>

int main() {
    struct Functor {
        void operator()() {
            std::cout << "Hello from functor!" << std::endl;
        }
    };
    std::thread t(Functor{});
    
    t.join();

    return 0;
}
```

5. 可调用对象为**类成员函数**：
```cpp
#include <thread>
#include <iostream>
#include <vector>

class MyClass {
private:
    int m_data;
    std::string m_name;
    
public:
    MyClass(int data, const std::string& name) 
	    : m_data(data), m_name(name) {}
    
    // 成员函数作为线程函数
    void memberFunction(int additional) {
        std::cout << "Member function: data=" << m_data 
                  << ", name=" << m_name 
                  << ", additional=" << additional << std::endl;
    }
    
    // 静态成员函数（不需要对象实例）
    static void staticMemberFunction(const std::string& message) {
        std::cout << "Static: " << message << std::endl;
    }
    
    // 在类内部启动线程
    void startInternalThread() {
        // 使用Lambda捕获this指针来访问成员变量
        std::thread t([this]() {
            std::cout << "Internal thread accessing: " << this->m_data << std::endl;
            this->m_data += 100;  // 修改成员变量
        });
        t.detach();  // 分离线程，让它在后台运行
    }
};

int main() {
    MyClass obj(42, "TestObject");
    
    // 方式1：成员函数作为线程函数
    // 语法：&类名::成员函数, 对象指针, 参数...
    std::thread t1(&MyClass::memberFunction, &obj, 100);
    t1.join();
    
    // 方式2：静态成员函数（用法和普通函数一样）
    std::thread t2(&MyClass::staticMemberFunction, "Hello Static");
    t2.join();
    
    // 方式3：在类内部启动线程
    obj.startInternalThread();
    
    // 给内部线程一点时间执行
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    
    return 0;
}
```

#### 线程中获取成员变量

1. **通过成员函数（推荐）**
```cpp
class Worker {
private:
    int m_counter;
    std::string m_status;
    
public:
    Worker() : m_counter(0), m_status("Idle") {}
    
    void doWork() {
        // 直接访问成员变量
        for (int i = 0; i < 5; ++i) {
            m_counter++;
            m_status = "Working " + std::to_string(i);
            std::cout << "Counter: " << m_counter << ", Status: " << m_status << std::endl;
            std::this_thread::sleep_for(std::chrono::milliseconds(500));
        }
    }
    
    void start() {
        std::thread t(&Worker::doWork, this);
        t.detach();
    }
};
```

2. **通过 Lambda 捕获 `this`**
```cpp
class Worker {
private:
    int m_counter;
    
public:
    Worker() : m_counter(0) {}
    
    void startWithLambda() {
        // Lambda捕获this指针来访问成员变量
        std::thread t([this]() {
            for (int i = 0; i < 5; ++i) {
                this->m_counter++;  // 通过this指针访问
                // 或者直接访问（Lambda捕获了this）
                std::cout << "Counter: " << m_counter << std::endl;
                std::this_thread::sleep_for(std::chrono::milliseconds(500));
            }
        });
        t.detach();
    }
};
```


### 线程管理

`std::thread` 有两种重要的管理线程的方法：`join()` 和 `detach()`，这两种方法决定了主线程（或创建线程的线程）如何管理与新创建的线程之间的关系。

1. **`.join()`**：会阻塞当前线程（通常是主线程或父线程），直到被调用的线程执行完毕。
	- **阻塞调用**：调用线程会等待目标线程完成
	- **同步操作**：确保线程按预期顺序执行
	- **资源清理**：线程完成后会自动清理资源
	- **一次性操作**：一个线程只能被 `join()` 一次
```cpp
#include <iostream>
#include <thread>
#include <chrono>

void worker(int id) {
    std::cout << "Thread " << id << " started\n";
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::cout << "Thread " << id << " finished\n";
}

int main() {
    std::cout << "Main thread started\n";
    
    std::thread t(worker, 1);
    
    std::cout << "Main thread waiting for worker to finish...\n";
    t.join();  // 主线程在这里阻塞，等待t完成
    
    std::cout << "Main thread continues after join()\n";
    return 0;
}
```

2. **`detach()`**：将线程与 `std::thread` 对象分离，让线程在后台独立运行。
	- **非阻塞调用**：立即返回，不等待线程完成
	- **独立运行**：线程在后台自主运行
	- **失去控制**：分离后无法再控制该线程
	- **自动清理**：线程结束后自动释放资源
```cpp
#include <iostream>
#include <thread>
#include <chrono>

void backgroundTask(int id) {
    for (int i = 0; i < 5; ++i) {
        std::cout << "Background task " << id << " - iteration " << i << "\n";
        std::this_thread::sleep_for(std::chrono::milliseconds(500));
    }
    std::cout << "Background task " << id << " completed\n";
}

int main() {
    std::cout << "Main thread started\n";
    
    std::thread t(backgroundTask, 1);
    t.detach();  // 分离线程，让它在后台运行
    
    std::cout << "Thread detached, main thread continues immediately\n";
    
    // 主线程继续执行其他工作
    for (int i = 0; i < 3; ++i) {
        std::cout << "Main thread working... " << i << "\n";
        std::this_thread::sleep_for(std::chrono::milliseconds(300));
    }
    
    std::cout << "Main thread finished\n";
    
    // 注意：分离的线程可能还在运行，但主线程已经结束
    // 在实际程序中，需要确保分离的线程在主线程结束前完成
    std::this_thread::sleep_for(std::chrono::seconds(3));
    
    return 0;
}
```


#### 重要规则

1. **规则1**：**线程必须被 `join()` 或 `detach()`**：在 `std::thread` 对象销毁前，必须调用 `join()` 或 `detach()`，否则会导致整个程序终止（包括主线程和所有其他线程）
```cpp
// 错误示例：线程未被 join 或 detach
void dangerousExample() {
    std::thread t([]{ 
        std::cout << "Running...\n"; 
    });
    // t 超出作用域，但既没有 join() 也没有 detach()
    // 程序会调用 std::terminate()！
}
```

```cpp
// 正确示例
void safeExample() {
    std::thread t([]{ 
        std::cout << "Running...\n"; 
    });
    t.join();  // 或者 t.detach()
}
```


2. **规则2**：**使用 RAII 确保线程安全**
```cpp
#include <thread>
#include <iostream>

class ThreadGuard {
private:
    std::thread& m_thread;
    
public:
    explicit ThreadGuard(std::thread& t) : m_thread(t) {}
    
    ~ThreadGuard() {
        if (m_thread.joinable()) {
            m_thread.join();  // 确保线程被 join
        }
    }
    
    // 禁止拷贝
    ThreadGuard(const ThreadGuard&) = delete;
    ThreadGuard& operator=(const ThreadGuard&) = delete;
};

void useThreadGuard() {
    std::thread t([]{
        std::this_thread::sleep_for(std::chrono::seconds(1));
        std::cout << "Thread completed\n";
    });
    
    ThreadGuard guard(t);  // 析构时会自动 join
    
    // 即使这里抛出异常，guard 的析构函数也会确保线程被 join
}  // guard 析构，自动调用 t.join()
```


3. **规则3**：**检查线程是否可 `join`**
```cpp
void checkJoinable() {
    std::thread t([]{ 
        std::cout << "Working...\n"; 
    });
    
    if (t.joinable()) {
        std::cout << "Thread is joinable\n";
        t.join();  // 或者 t.detach()
    }
    
    // join() 或 detach() 后，线程不再 joinable
    if (!t.joinable()) {
        std::cout << "Thread is no longer joinable\n";
    }
}
```



### 数据竞争保护

```cpp
#include <thread>
#include <mutex>

class SafeCounter {
private:
    int m_count;
    mutable std::mutex m_mutex;
    
public:
    SafeCounter() : m_count(0) {}
    
    void increment() {
        std::lock_guard<std::mutex> lock(m_mutex);
        m_count++;
    }
    
    int getCount() const {
        std::lock_guard<std::mutex> lock(m_mutex);
        return m_count;
    }
    
    void incrementFromThread() {
        std::thread t([this]() {
            for (int i = 0; i < 1000; ++i) {
                this->increment();  // 线程安全地增加计数
            }
        });
        t.join();
    }
};
```


## 高级用法

### 线程池

在多线程编程中，频繁创建和销毁线程会带来巨大的性能开销，线程池通过预先创建一批线程并复用它们执行任务，有效降低了线程管理成本，提升了程序的响应速度和资源利用率。

> [!tip]- 频繁创建和销毁线程的巨大的性能开销分析
> 1. **系统调用开销**：每次创建线程都需要系统调用，从用户态->内核态
> 	- 上下文切换：用户态到内核态的切换
> 	- 参数验证：内核需要验证参数合法性
> 	- 系统调用表查找：查找对应的系统调用处理函数
> 2. **内存分配开销**：每个线程都需要独立的栈空间
> 	- 默认栈空间大小：通常为 2-8MB，Linux 通常 8MB，Windows 默认 1MB
> 3. **初始化开销**：线程创建时，内部初始化开始工作
> 	- 分配和初始化栈空间
> 	- 设置线程上下文
> 	- 初始化线程局部存储(TLS)
> 	- 设置信号掩码
> 	- 初始化浮点状态
> 	- 注册到调度器
> 	- 建立异常处理框架
> 4. **缓存局部性影响**：频繁线程切换导致缓存失效
> ```cpp
> void cache_locality_issue() {
 >   // 频繁线程切换导致缓存失效
 >   for (int i = 0; i < 100; i++) {
 >       std::thread t([i] {
>            // 每个新线程的栈在不同内存位置
>            // 导致CPU缓存频繁失效
 >           process_data(i);
>        });
 >       t.join();
 >   }
>    
 >   // vs 线程池：相同线程处理多个任务，缓存友好
>}
> ```

> [!info] 线程创建销毁的主要 CPU 开销对比
>1. 系统调用            ~1000-5000 cycles
>2. 上下文保存恢复      ~100-1000 cycles  
>3. 内存分配            ~1000-10000 cycles
>4. 缓存失效           ~100-1000 cycles
>5. TLB刷新            ~100-500 cycles

#### 基本组成

```cpp
#include <vector>
#include <queue>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <future>
#include <functional>
#include <stdexcept>
#include <atomic>

class ThreadPool {
public:
    explicit ThreadPool(size_t threads)
	    : stop(false)
    {
        for(size_t i = 0; i < threads; ++i) {
            workers.emplace_back([this] {
                for(;;) {
                    std::function<void()> task;
                    {
                        std::unique_lock<std::mutex> lock(this->queue_mutex);
                        this->condition.wait(lock, [this] { 
                            return this->stop || !this->tasks.empty(); 
                        });
                        if(this->stop && this->tasks.empty())
                            return;
                        task = std::move(this->tasks.front());
                        this->tasks.pop();
                    }
                    task();
                }
            });
        }
    }
    
    ~ThreadPool()
    {
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            stop = true;
        }
        condition.notify_all();
        for(std::thread &worker : workers)
            worker.join();
    }
    
    template<class F, class... Args>
    auto enqueue(F&& f, Args&&... args) 
        -> std::future<typename std::result_of<F(Args...)>::type> 
    {
        using return_type = typename std::result_of<F(Args...)>::type;
        
        auto task = std::make_shared<std::packaged_task<return_type()>>(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );
        
        std::future<return_type> res = task->get_future();
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            if(stop)
                throw std::runtime_error("enqueue on stopped ThreadPool");
            tasks.emplace([task](){ (*task)(); });
        }
        condition.notify_one();
        return res;
    }

private:
    std::vector<std::thread> workers;
    std::queue<std::function<void()>> tasks;
    
    std::mutex queue_mutex;
    std::condition_variable condition;
    std::atomic<bool> stop;
};
```

#### C++17/20 现代化版本

```cpp
#include <vector>
#include <queue>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <future>
#include <functional>
#include <atomic>

class ThreadPool {
public:
    explicit ThreadPool(size_t num_threads = std::thread::hardware_concurrency()) 
        : stop(false)
	{
        start(num_threads);
    }
    
    ~ThreadPool()
    {
        shutdown();
    }
    
    // 禁止拷贝和移动
    ThreadPool(const ThreadPool&) = delete;
    ThreadPool& operator=(const ThreadPool&) = delete;
    
    template<typename F, typename... Args>
    auto submit(F&& f, Args&&... args) 
	    -> std::future<std::invoke_result_t<F, Args...>> 
    {
        using return_type = std::invoke_result_t<F, Args...>;
        
        auto task = std::make_shared<std::packaged_task<return_type()>>(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );
        
        std::future<return_type> result = task->get_future();
        {
            std::unique_lock lock(mutex_);
            if(stop) {
                throw std::runtime_error("Submit on stopped ThreadPool");
            }
            tasks_.emplace([task](){ (*task)(); });
        }
        condition_.notify_one();
        return result;
    }
    
    void shutdown()
    {
        {
            std::unique_lock lock(mutex_);
            stop = true;
        }
        condition_.notify_all();
        for(auto& thread : threads_) {
            if(thread.joinable()) {
                thread.join();
            }
        }
    }

private:
    void start(size_t num_threads) 
    {
        for(size_t i = 0; i < num_threads; ++i) {
            threads_.emplace_back([this] {
                while(true) {
                    std::function<void()> task;
                    {
                        std::unique_lock lock(mutex_);
                        condition_.wait(lock, [this] {
                            return stop || !tasks_.empty();
                        });
                        
                        if(stop && tasks_.empty()) {
                            return;
                        }
                        
                        task = std::move(tasks_.front());
                        tasks_.pop();
                    }
                    task();
                }
            });
        }
    }
    
    std::vector<std::thread> threads_;
    std::queue<std::function<void()>> tasks_;
    std::mutex mutex_;
    std::condition_variable condition_;
    std::atomic<bool> stop;
};
```

#### 适用场景

1. **高并发服务器**
```cpp
// Web服务器处理HTTP请求
void handle_http_request(int client_socket) {
    // 处理HTTP请求的逻辑
}

// 使用线程池处理并发连接
ThreadPool pool(16);
while(true) {
    int client_socket = accept_connection();
    pool.submit(handle_http_request, client_socket);
}
```

2. **并行计算和数据处理**
```cpp
// 并行处理大数据集
void process_data_chunk(const std::vector<int>& chunk, int start, int end) {
    for(int i = start; i < end; ++i) {
        // 处理数据
    }
}

ThreadPool pool;
std::vector<std::future<void>> futures;
std::vector<int> data(1000000);

// 分块并行处理
size_t chunk_size = data.size() / 8;
for(size_t i = 0; i < 8; ++i) {
    size_t start = i * chunk_size;
    size_t end = (i == 7) ? data.size() : (i + 1) * chunk_size;
    futures.push_back(pool.submit(process_data_chunk, 
                                 std::ref(data), start, end));
}

// 等待所有任务完成
for(auto& future : futures) {
    future.get();
}
```

3. **异步I/O操作**
```cpp
// 异步文件操作
std::future<std::string> read_file_async(ThreadPool& pool, 
                                        const std::string& filename) {
    return pool.submit([](const std::string& file) -> std::string {
        std::ifstream stream(file);
        return std::string(std::istreambuf_iterator<char>(stream),
                          std::istreambuf_iterator<char>());
    }, filename);
}
```

4. **定时任务调度**
```cpp
class TaskScheduler {
    ThreadPool pool;
public:
    void schedule_recurring(std::function<void()> task, 
                           std::chrono::milliseconds interval) {
        pool.submit([task, interval] {
            while(true) {
                std::this_thread::sleep_for(interval);
                task();
            }
        });
    }
};
```