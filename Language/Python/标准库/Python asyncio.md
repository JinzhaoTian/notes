Python 的 `asyncio` 是用于编写并发代码的标准库，提供了完整的事件循环、协程、任务等基础设施，从 Python 3.4 引入，3.5+ 成为标准库核心。

`asyncio` 是标准库，但实际使用时通常需要配合第三方异步库来完成具体工作。
```python
# 这是标准库
import asyncio

# 第三方的异步库（基于标准库）
import aiohttp      # 异步 HTTP 客户端
import asyncpg      # 异步 PostgreSQL 驱动
import aiomysql     # 异步 MySQL 驱动
```


## 核心组件

1. **事件循环**（Event Loop）
```python
import asyncio

# 获取事件循环
loop = asyncio.get_event_loop()

# 运行协程直到完成
loop.run_until_complete(some_coroutine())

# 永久运行事件循环
loop.run_forever()

# Python 3.7+ 推荐使用 asyncio.run()
asyncio.run(main())  # 自动创建、运行、关闭循环
```

2. **协程**（Coroutine）
```python
async def my_coroutine():
    print("开始")
    await asyncio.sleep(1)  # 模拟异步操作
    print("结束")
    return "结果"

# 协程对象不会自动执行
coro = my_coroutine()

# 需要事件循环驱动
result = await coro  # 或 asyncio.run(coro)
```

> [!caution] 协程 vs 生成器
> 异步的底层基于[生成器（Generator）](../原理/Python%20Generator.md)实现：
> ```python
> # 早期 Python 3.4 使用生成器实现
> @asyncio.coroutine
> def old_style():
>    yield from asyncio.sleep(1)
>
> # Python 3.5+ 使用 async/await 语法糖
> async def new_style():
>     await asyncio.sleep(1)
>
> # 本质上都是生成器对象
> print(type(new_style()))  # <class 'coroutine'>
> ```

3. **任务**（Task）：用于并发运行多个协程
```python
async def main():
    # 创建任务（自动调度）
    task1 = asyncio.create_task(my_coroutine())
    task2 = asyncio.create_task(my_coroutine())
    
    # 等待所有任务完成
    await asyncio.gather(task1, task2)
    
    # 或者等待第一个完成
    done, pending = await asyncio.wait(
        [task1, task2], 
        return_when=asyncio.FIRST_COMPLETED
    )
```

4. **Future 对象**：表示异步操作的最终结果
```python
async def set_future():
    future = asyncio.Future()
    
    # 稍后设置结果
    asyncio.create_task(set_result_after_delay(future))
    
    # 等待结果
    result = await future
    return result

async def set_result_after_delay(future):
    await asyncio.sleep(1)
    future.set_result("完成")
```

## 常用 API

1. **并发控制**
```python
import asyncio

async def worker(name, delay):
    await asyncio.sleep(delay)
    return f"{name} 完成"

async def main():
    # gather: 并发执行并收集结果
    results = await asyncio.gather(
        worker("A", 1),
        worker("B", 2),
        worker("C", 1.5),
        return_exceptions=True  # 捕获异常
    )
    
    # wait: 更灵活的控制
    tasks = [worker(f"任务{i}", i) for i in range(5)]
    done, pending = await asyncio.wait(
        tasks,
        timeout=3,
        return_when=asyncio.FIRST_COMPLETED
    )
    
    # as_completed: 逐个获取完成的结果
    for coro in asyncio.as_completed([worker("X", 1), worker("Y", 2)]):
        result = await coro
        print(result)
```

2. **同步原语**
```python
# 锁
lock = asyncio.Lock()
async def safe_operation():
    async with lock:
        # 临界区代码
        pass

# 信号量（限制并发数）
semaphore = asyncio.Semaphore(5)
async def limited_operation():
    async with semaphore:
        # 最多5个并发
        pass

# 事件
event = asyncio.Event()
async def waiter():
    await event.wait()  # 等待事件
    print("事件已触发")

async def setter():
    await asyncio.sleep(1)
    event.set()  # 触发事件

# 队列
queue = asyncio.Queue(maxsize=10)
await queue.put(item)
item = await queue.get()
```

## 使用示例

1. **并发网络请求**
```python
import asyncio
import aiohttp

async def fetch_url(session, url):
    async with session.get(url) as response:
        return await response.text()

async def fetch_all(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
    return results

# 使用
urls = ['http://example.com', 'http://example.org', 'http://example.net']
results = asyncio.run(fetch_all(urls))
```

2. **生产者-消费者模式**
```python
import asyncio
import random

async def producer(queue, name):
    for i in range(5):
        item = f"{name}-{i}"
        await queue.put(item)
        print(f"生产: {item}")
        await asyncio.sleep(random.random())
    await queue.put(None)  # 结束信号

async def consumer(queue, name):
    while True:
        item = await queue.get()
        if item is None:
            break
        print(f"消费: {item} by {name}")
        await asyncio.sleep(random.random())

async def main():
    queue = asyncio.Queue(maxsize=10)
    
    # 启动生产者和消费者
    producers = [asyncio.create_task(producer(queue, f"P{i}")) for i in range(2)]
    consumers = [asyncio.create_task(consumer(queue, f"C{i}")) for i in range(3)]
    
    await asyncio.gather(*producers)
    await queue.join()  # 等待队列清空
    
    # 停止消费者
    for _ in consumers:
        await queue.put(None)
    await asyncio.gather(*consumers)

asyncio.run(main())
```

3. **超时控制**
```python
async def slow_operation():
    await asyncio.sleep(10)
    return "结果"

async def main():
    try:
        # 等待最多5秒
        result = await asyncio.wait_for(slow_operation(), timeout=5)
    except asyncio.TimeoutError:
        print("操作超时")
    
    # 带超时的 gather
    try:
        results = await asyncio.wait_for(
            asyncio.gather(slow_operation(), slow_operation()),
            timeout=3
        )
    except asyncio.TimeoutError:
        print("部分任务超时")
        # 取消未完成的任务
        for task in asyncio.all_tasks():
            if task is not asyncio.current_task():
                task.cancel()
```

## 最佳实践

1. **使用 `asyncio.run()`**
```python
# ✅ 推荐
async def main():
    # ...
    pass

asyncio.run(main())

# ❌ 避免手动管理循环
loop = asyncio.get_event_loop()
try:
    loop.run_until_complete(main())
finally:
    loop.close()
```

2. **避免阻塞操作**
```python
# ❌ 错误：阻塞事件循环
async def bad():
    time.sleep(5)  # 阻塞！
    
# ✅ 正确：使用异步版本
async def good():
    await asyncio.sleep(5)  # 非阻塞

# 对于 CPU 密集型操作
async def cpu_intensive():
    # 在线程池中运行
    result = await asyncio.to_thread(cpu_bound_function, arg)
    return result
```

3. **正确处理异常**
```python
async def main():
    tasks = [asyncio.create_task(may_fail()) for _ in range(5)]
    
    # 方法1: gather 捕获异常
    results = await asyncio.gather(*tasks, return_exceptions=True)
    
    # 方法2: 使用 try/except
    try:
        await asyncio.gather(*tasks)
    except Exception as e:
        print(f"发生异常: {e}")
        # 取消其他任务
        for task in tasks:
            task.cancel()
```

4. 