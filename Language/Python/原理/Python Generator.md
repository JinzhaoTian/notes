生成器（Generator）是 Python 中一种特殊的迭代器，它允许你**在迭代过程中动态生成值**，而不是一次性在内存中存储所有值，其核心特点是使用 `yield` 关键字，函数遇到 `yield` 时会暂停执行并返回一个值，下次调用时从暂停处继续。

> [!caution] 为什么要用生成器？
> |对比|普通列表|生成器|
> |---|---|---|
> |内存|一次性存储所有元素，占用大|每次只生成一个值，内存占用极小|
> |速度|创建时已生成所有数据|惰性求值，按需生成|
> |适用场景|数据量小，需要多次访问|数据量大（如读取大文件、无限序列）|
> 
> 如：生成 1000 万个数字
> ```python
> # 列表：占用大量内存
> nums = [i for i in range(10000000)]  # 立即创建 1000 万个整数
> 
> # 生成器：几乎不占内存
> nums_gen = (i for i in range(10000000))  # 只是定义了一个生成规则
> ```

> [!tip]
> 生成器让你用**写普通函数的语法**，实现**迭代器的行为**，同时享受**内存友好**的好处。

## 底层原理

Python 解释器在运行时：
1. 在编译时识别到 `yield` 关键字，将函数标记为生成器函数
2. 调用时不执行函数体，而是返回一个**生成器对象**（`PyGenObject`）
3. 生成器对象保存了：
    - 栈帧（`gi_frame`）- 保存执行状态
    - 代码对象（`gi_code`）
    - 挂起状态

生成器有自己独立的栈帧，这意味着它可以在堆上保存执行状态，这是底层机制支持的，不是简单的语法转换：
1. Python 解释器对生成器有**原生支持**（专门的字节码、对象类型）
2. 生成器保存**完整的执行栈帧**，可以实现暂停/恢复
3. 它是 Python **协程系统的基石**（async/await 底层就是基于生成器）
4. 支持 `send()`/`throw()`/`close()` 等高级协程特性

生成器是真的**底层对象**：
```python
import dis
import types

def my_gen():
    yield 1
    yield 2

# 1. 查看字节码 - 有专门的 YIELD_VALUE 指令
print(dis.dis(my_gen))
# 输出包含：
#  2           0 LOAD_CONST               1 (1)
#              2 YIELD_VALUE               # 专门的 yield 指令
#              4 POP_TOP

# 2. 类型检查 - 是独立的类型
print(type(my_gen))           # <class 'function'>
g = my_gen()
print(type(g))                # <class 'generator'>
print(isinstance(g, types.GeneratorType))  # True

# 3. 查看生成器特有的属性
print(dir(g))
# 包含 gi_frame, gi_code, gi_running 等底层属性
```


## 创建

1. **方式一**：**生成器函数（使用 `yield`）**
```python
def count_up_to(n):
    i = 0
    while i < n:
        yield i      # 返回 i，并暂停
        i += 1

# 调用返回生成器对象，不会立即执行
counter = count_up_to(5)

for num in counter:
    print(num)  # 输出 0,1,2,3,4
```

2. **方式二**：**生成器表达式（类似列表推导式，但用圆括号）**
```python
# 列表推导式
squares_list = [x**2 for x in range(10)]   # 立即生成列表

# 生成器表达式
squares_gen = (x**2 for x in range(10))    # 返回生成器对象

print(next(squares_gen))  # 0
print(next(squares_gen))  # 1
```

## 使用

1. **方法一**：**`next()` 手动获取下一个值**
```python
gen = (x for x in range(3))
print(next(gen))  # 0
print(next(gen))  # 1
print(next(gen))  # 2
print(next(gen))  # 抛出 StopIteration 异常
```

2. **方法二**：**`for` 循环（自动处理 StopIteration）**
```python
gen = (x for x in range(3))
for val in gen:
    print(val)  # 0, 1, 2
```

3. **方法三**：**转换为列表（会消耗生成器）**
```python
gen = (x for x in range(5))
lst = list(gen)  # [0, 1, 2, 3, 4]
# 此时生成器已耗尽，再次调用 list(gen) 得到 []
```


## 执行流程

```python
def my_gen():
    print("开始执行")
    yield 1
    print("继续执行")
    yield 2
    print("结束")

g = my_gen()
print(next(g))  # 打印"开始执行"，然后打印 1
print(next(g))  # 打印"继续执行"，然后打印 2
print(next(g))  # 打印"结束"，然后抛出 StopIteration
```

> [!caution] 关键
> 生成器函数内部的代码是**分段执行**的，每次 `yield` 就像是一个断点。



## 应用场景

1. **处理大文件**（避免内存爆炸）
```python
def read_large_file(file_path):
    with open(file_path, 'r') as f:
        for line in f:
            yield line.strip()  # 逐行返回，不一次性加载全部

# 使用
for line in read_large_file('huge.log'):
    process(line)  # 处理每一行
```

2. **生成无限序列**
```python
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

fib = fibonacci()
for _ in range(10):
    print(next(fib))  # 0,1,1,2,3,5,8,13,21,34
```

3. **管道式数据处理**
```python
# 链式处理数据，每个环节都是惰性的
numbers = (x for x in range(100))
evens = (x for x in numbers if x % 2 == 0)
squares = (x**2 for x in evens)

for val in squares:
    if val > 100:
        break
    print(val)
```

## 高级特性

1. **`send()`** - 向生成器发送值
```python
def accumulator():
    total = 0
    while True:
        value = yield total
        if value is not None:
            total += value

acc = accumulator()
next(acc)          # 启动生成器，返回 0
print(acc.send(5)) # 发送 5，total 变成 5，返回 5
print(acc.send(3)) # 发送 3，total 变成 8，返回 8
```

2. **`throw()`** - 向生成器抛出异常
```python
def gen():
    try:
        yield 1
        yield 2
    except ValueError:
        yield 'caught'

g = gen()
print(next(g))       # 1
print(g.throw(ValueError))  # 'caught'
```

3. **`close()`** - 关闭生成器
```python
def gen():
    yield 1
    yield 2

g = gen()
print(next(g))  # 1
g.close()
print(next(g))  # 抛出 StopIteration（生成器已关闭）
```

