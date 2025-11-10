Folly（Facebook Open Source Library）是 Facebook（Meta） 开源的一个 C++ 库，提供了大量高性能、实用的组件和工具，是对 C++ 标准库的补充，特别针对大规模、高性能的应用场景设计。

## 主要组件

1. **核心数据结构**
	- `fbstring`：高性能字符串类，比 `std::string` 更高效
	- `FBVector`：优化的向量容器，性能优于 `std::vector`
	- `AtomicHashMap`：线程安全的哈希映射
	- `EvictingCacheMap`：带淘汰策略的缓存映射

2. **并发编程工具**
	- `AtomicStruct`：原子结构体操作
	- `MPMCQueue`：多生产者多消费者队列
	- `Baton`：轻量级同步原语
	- `Synchronized`：简化同步访问的包装器

3. **异步编程**
	- `Future/Promise`：功能强大的异步编程框架
	- `Executors`：执行器框架，管理异步任务执行
	- `Fibers`：用户态协程支持

4. **实用工具**
	- `Format`：类型安全的格式化库
	- `Range`：范围操作工具
	- `File`：文件操作工具
	- `JSON`：高性能 JSON 解析和序列化

5. **性能优化工具**
	- `MicroLock`：极小的锁实现
	- `PackedSyncPtr`：压缩的指针+同步原语
	- `ThreadCachedInt`：线程本地缓存的计数器

## 使用示例
 
1. **`Future` / `Promise`**：
```cpp
#include <folly/futures/Future.h>
#include <folly/executors/ThreadedExecutor.h>

folly::Promise<int> p;
folly::Future<int> f = p.getFuture();

f.then([](int value) {
    std::cout << "Got value: " << value << std::endl;
    return value + 1;
});

p.setValue(42);
```

2. **`fbstring`**：
```cpp
#include <folly/FBString.h>

folly::fbstring str = "Hello, Folly!";
str += " This is efficient!";
```

3. **`Synchronized`**：
```cpp
#include <folly/Synchronized.h>

folly::Synchronized<std::vector<int>> vec;

{
    auto lockedVec = vec.lock();
    lockedVec->push_back(42);
    lockedVec->push_back(43);
} // 自动释放锁
```