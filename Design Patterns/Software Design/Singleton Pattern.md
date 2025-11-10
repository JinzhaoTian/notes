![](../imgs/DesignPattern-Singleton.png)

> [!info] **什么时候应该用Singleton？**
> 实际上，很多程序，尤其是 Web 程序，大部分服务类都应该被视作 Singleton，如果全部按Singleton 的写法写，会非常麻烦，所以，通常是通过约定让框架来实例化这些类，保证只有一个实例，调用方自觉**通过框架获取实例**而不是 `new` 操作符。
> 
> 因此，除非确有必要，否则 Singleton 模式一般以“约定”为主，不会刻意实现它，如 Java 中的 Spring。


## 实现方式

1. 实现一：线程不安全（懒汉模式）
```C++
class Singleton{
    private:
        Singleton();  // 构造函数需要私有化，延迟初始化
        Singleton(const Singleton& other);
        static Singleton* m_instance;
    
    public:
        static Singleton* getInstance(){
            if (m_instance == nullptr) {
                m_instance = new Singleton();
            }
            return m_instance;
        }

};
```

2. 实现二：多线程加锁版
```C++
// 懒汉模式，线程安全版本，但锁的代价过高
Singleton* Singleton::getInstance() {
    Lock lock; //伪代码 加锁
    if (m_instance == nullptr) {
        m_instance = new Singleton();
    }
    return m_instance;
}


//双检查锁，但由于内存读写reorder不安全
Singleton* Singleton::getInstance() {
    //先判断是不是初始化了，如果初始化过，就再也不会使用锁了
    if(m_instance == nullptr){
        Lock lock; //伪代码
        if (m_instance == nullptr) {
            m_instance = new Singleton();
        }
    }
    return m_instance;
}
```

3. 实现三：C++11 跨平台版
```C++
//C++ 11版本之后的跨平台实现 
// atomic c++11中提供的原子操作
std::atomic<Singleton*> Singleton::m_instance;
std::mutex Singleton::m_mutex;

/*
* std::atomic_thread_fence(std::memory_order_acquire); 
* std::atomic_thread_fence(std::memory_order_release);
* 这两句话可以保证他们之间的语句不会发生乱序执行。
*/
Singleton* Singleton::getInstance() {
    Singleton* tmp = m_instance.load(std::memory_order_relaxed);
    std::atomic_thread_fence(std::memory_order_acquire);//获取内存fence
    if (tmp == nullptr) {
        std::lock_guard<std::mutex> lock(m_mutex);
        tmp = m_instance.load(std::memory_order_relaxed);
        if (tmp == nullptr) {
            tmp = new Singleton;
            std::atomic_thread_fence(std::memory_order_release);//释放内存fence
            m_instance.store(tmp, std::memory_order_relaxed);
        }
    }
    return tmp;
}
```




## 双重检查锁定

在 C++11 之前，为了实现线程安全的延迟初始化，通常双重检查锁定（Double-Checked Locking Pattern, DCLP）。

```cpp
// 旧式、复杂且在现代C++中已不必要的实现
class OldSingleton {
public:
    static OldSingleton* getInstance() {
        if (instance == nullptr) { // 第一次检查（无锁，快速路径）
            std::lock_guard<std::mutex> lock(mutex); // 加锁
            if (instance == nullptr) { // 第二次检查（加锁后）
                instance = new OldSingleton();
            }
        }
        return instance;
    }
    // ... 其他部分类似，但需要定义 static OldSingleton* instance 和 static std::mutex mutex;

private:
    static OldSingleton* instance;
    static std::mutex mutex;
    // ... 构造函数等
};

// 必须在类外初始化静态成员
OldSingleton* OldSingleton::instance = nullptr;
std::mutex OldSingleton::mutex;
```

这种方式不仅代码冗长，而且在没有内存模型保证的旧标准下，由于指令重排（`new` 操作分配内存、调用构造函数、赋值给指针这三个步骤可能被重排），可能导致其他线程看到一个已分配内存但未构造完成的对象，从而引发未定义行为。

解决这个问题需要使用特定平台的内存屏障（Memory Barrier），非常复杂。



## Meyers' Singleton（推荐）（首选）

Meyers' Singleton 是以其提出者 Scott Meyers 命名的，它是一种在 C++ 中实现单例模式的最佳方法，这种方法巧妙地利用了 C++ 标准中关于静态变量的特性，实现了**线程安全**、**延迟初始化**和**简洁性**。

### 核心思想

**在函数内部**定义一个静态局部变量，并返回该变量的引用，这个静态局部变量只会被初始化一次。


### 经典实现

```cpp
class Singleton {
public:
    // 删除拷贝构造函数和赋值操作符，确保唯一性
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

    // 全局访问点
    static Singleton& getInstance() {
        static Singleton instance; // 线程安全的延迟初始化发生在这里
        return instance;
    }

    // 一个示例成员函数
    void doSomething() {
        // ...
    }

private:
    // 将构造函数私有化，防止外部创建实例
    Singleton() {
        // 初始化代码
    }

    // 析构函数（可根据需要设为公有或私有）
    ~Singleton() {
        // 清理代码
    }
};

// 使用方法：
Singleton::getInstance().doSomething();
```

### 为什么 Meyers' Singleton 如此优秀？

Meyers' Singleton 解决了传统单例实现方式的几个主要痛点：

1. **线程安全（Thread Safety）**：
    - 在 C++11 及以后的标准中，标准明确规定：**静态局部变量的初始化是线程安全的**。
    - 编译器会在地下自动插入类似的代码来保证 `instance` 只被初始化一次，其他线程在初始化完成前会被阻塞。这意味着你不需要自己手动使用互斥锁（`std::mutex`）或双重检查锁定模式（DCLP）等容易出错的技巧来保证线程安全。

2. **延迟初始化（Lazy Initialization）**：
    - 单例实例只会在 `getInstance()` 函数**第一次被调用时**才创建。
    - 这**避免了程序启动时就初始化所有单例带来的启动开销**，如果某个单例在整个程序运行中都未被使用，那么它就永远不会被创建，节省了资源。

3. **简洁性与可靠性（Simplicity & Reliability）**：
    - 代码非常简洁易懂，没有复杂的锁机制或空指针检查。
    - 自动处理了析构，当程序结束时，`instance` 的析构函数会被自动调用。而一些其他实现（如用 `new` 在堆上创建实例）如果不手动 `delete` 则会导致内存泄漏。

4. **解决静态初始化顺序问题**：
    - 在**跨编译单元的静态变量初始化中，其顺序是未定义的**，如果两个单例在初始化时相互依赖，会导致问题。
    - Meyers' Singleton 通过将实例定义为**函数内的静态变量**，确保了它在函数第一次被调用时（此时所有静态初始化肯定已经完成）才初始化，从而避免了这个问题。