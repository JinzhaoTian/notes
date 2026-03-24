Python `threading` 模块是用于**多线程编程**的标准库，提供了创建和管理线程的高级接口。它是对底层 `_thread` 模块的封装，让多线程编程更加简单和安全。

> [!caution] Python 线程重要特性
> 由于[全局解释器锁（GIL）](../原理/Python%20GIL.md)（CPython 解释器中的互斥锁）的存在，因此：
> - **同一时刻只允许一个线程执行 Python 字节码**，意味着 Python 的线程无法实现真正的并行计算
> - 对于 CPU 密集型任务（复杂计算、图像处理），应使用多进程
> - 对于 I/O 密集型任务（网络请求、文件读写、数据库操作），多线程仍然有效


## 核心组件

1. **Thread** - 线程对象
```python
import threading
import time

# 创建线程的两种方式

# 方式一：传递目标函数
def print_numbers():
    for i in range(5):
        print(f"数字: {i}")
        time.sleep(0.5)

t = threading.Thread(target=print_numbers)
t.start()  # 启动线程
t.join()   # 等待线程结束

# 方式二：继承 Thread 类
class MyThread(threading.Thread):
    def __init__(self, name):
        super().__init__()
        self.name = name
    
    def run(self):
        for i in range(3):
            print(f"{self.name}: {i}")
            time.sleep(0.5)

t = MyThread("Worker")
t.start()
```


```python
# 线程的常用属性和方法
t = threading.Thread(target=func, name="MyThread", daemon=True)

t.start()           # 启动线程
t.join(timeout=2)   # 等待线程结束，最多等待2秒
t.is_alive()        # 检查线程是否存活
t.name              # 线程名称
t.ident             # 线程ID
t.daemon            # 是否为守护线程
threading.current_thread()  # 获取当前线程对象
threading.active_count()    # 活动线程数量
threading.enumerate()       # 所有活动线程列表
```

2. **Lock** - 锁机制，防止多个线程同时访问共享资源。
```python
import threading
counter = 0
lock = threading.Lock()
def increment():
    global counter
    for _ in range(100000):
        # 方式1：使用上下文管理器
        with lock:
            counter += 1
        
        # 方式2：手动获取和释放
        # lock.acquire()
        # try:
        #     counter += 1
        # finally:
        #     lock.release()
threads = []
for _ in range(5):
    t = threading.Thread(target=increment)
    threads.append(t)
    t.start()
for t in threads:
    t.join()
print(f"计数器值: {counter}")  # 500000
```

3. **RLock** - 可重入锁，允许同一线程多次获取锁，避免死锁。
```python
import threading
rlock = threading.RLock()
def recursive_func(n):
    with rlock:
        if n > 0:
            print(f"递归层 {n} - 持有锁")
            recursive_func(n - 1)  # 同一线程再次获取锁
recursive_func(3)  # 不会死锁
```

4. **Semaphore** - 信号量，控制同时访问资源的线程数量。
```python
import threading
import time

# 限制同时只有3个线程可以执行
semaphore = threading.Semaphore(3)
def limited_task(name):
    with semaphore:
        print(f"{name} 获得许可，开始执行")
        time.sleep(2)
        print(f"{name} 执行完成，释放许可")
threads = []
for i in range(10):
    t = threading.Thread(target=limited_task, args=(f"线程{i}",))
    threads.append(t)
    t.start()
for t in threads:
    t.join()
```

5. **Event** - 事件，用于线程间通信，一个线程等待另一个线程发出信号。
```python
import threading
import time
event = threading.Event()
def waiter():
    print("等待者: 等待事件...")
    event.wait()  # 阻塞直到事件被设置
    print("等待者: 收到事件，继续执行")
def setter():
    print("设置者: 3秒后设置事件...")
    time.sleep(3)
    event.set()  # 设置事件，唤醒等待线程
    print("设置者: 事件已设置")
t1 = threading.Thread(target=waiter)
t2 = threading.Thread(target=setter)
t1.start()
t2.start()
t1.join()
t2.join()
```

6. **Condition** - 条件变量，更复杂的同步机制，允许线程等待特定条件满足。
```python
import threading
import time
class ProducerConsumer:
    def __init__(self):
        self.condition = threading.Condition()
        self.items = []
    
    def produce(self):
        with self.condition:
            for i in range(5):
                self.items.append(i)
                print(f"生产: {i}")
                self.condition.notify()  # 通知消费者
                time.sleep(1)
    
    def consume(self):
        with self.condition:
            while len(self.items) < 3:  # 等待至少3个商品
                print("消费者等待...")
                self.condition.wait()
            
            # 消费3个商品
            for _ in range(3):
                item = self.items.pop(0)
                print(f"消费: {item}")
pc = ProducerConsumer()
producer = threading.Thread(target=pc.produce)
consumer = threading.Thread(target=pc.consume)
consumer.start()  # 消费者先启动，会等待
producer.start()
producer.join()
consumer.join()
```

7. **Timer** - 定时器，延迟执行任务的线程。
```python
import threading
def delayed_task():
    print("延迟任务执行了")
# 3秒后执行
timer = threading.Timer(3, delayed_task)
timer.start()
# 可以取消定时器
# timer.cancel()
```

8. **Barrier** - 屏障，让多个线程等待直到所有线程都到达屏障点。
```python
import threading
import time
barrier = threading.Barrier(3)  # 需要3个线程到达
def worker(name):
    print(f"{name} 开始工作")
    time.sleep(2)
    print(f"{name} 到达屏障")
    barrier.wait()  # 等待其他线程
    print(f"{name} 继续执行")
threads = []
for i in range(3):
    t = threading.Thread(target=worker, args=(f"线程{i}",))
    threads.append(t)
    t.start()
for t in threads:
    t.join()
```

9. **Queue** - 线程安全队列，虽然不是 threading 模块的一部分，但常与 threading 配合使用。
```python
from queue import Queue
import threading
# 创建线程安全队列
q = Queue(maxsize=10)
def producer():
    for i in range(20):
        q.put(i)
        print(f"生产: {i}")
    q.put(None)  # 结束信号
def consumer():
    while True:
        item = q.get()
        if item is None:
            break
        print(f"消费: {item}")
        q.task_done()
p = threading.Thread(target=producer)
c = threading.Thread(target=consumer)
p.start()
c.start()
p.join()
c.join()
```




## 高级特性

1. **线程局部变量**
```python
import threading
# 每个线程独有的数据
local_data = threading.local()
def process():
    local_data.value = threading.current_thread().name
    print(f"{threading.current_thread().name}: {local_data.value}")
for i in range(3):
    t = threading.Thread(target=process, name=f"线程{i}")
    t.start()
```

2. **异常处理**
```python
import threading
import sys
def thread_with_exception():
    try:
        raise ValueError("线程内异常")
    except Exception as e:
        print(f"捕获到异常: {e}")
        sys.exc_info()
t = threading.Thread(target=thread_with_exception)
t.start()
t.join()
```


## 使用示例

1. **方式一**：**创建 Thread 实例**
```python
import threading
import time

# 方式1：创建 Thread 实例
def worker(name, delay):
    """线程执行的任务"""
    for i in range(3):
        print(f"线程 {name}: 第 {i+1} 次执行")
        time.sleep(delay)

# 创建线程
t1 = threading.Thread(target=worker, args=("A", 1))
t2 = threading.Thread(target=worker, args=("B", 0.5))

# 启动线程
t1.start()
t2.start()

# 等待线程结束
t1.join()
t2.join()

print("所有线程执行完毕")
```

2. **方式二**：**继承 Thread 类**
```python
import threading
import time

class MyThread(threading.Thread):
    def __init__(self, name):
        super().__init__()
        self.name = name
    
    def run(self):
        """重写 run 方法"""
        for i in range(3):
            print(f"线程 {self.name}: {i}")
            time.sleep(1)

t = MyThread("Worker")
t.start()
t.join()
```