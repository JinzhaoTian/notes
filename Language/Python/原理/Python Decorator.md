Python 装饰器（Decorator）本质上是一个**高阶函数**，它允许在不修改原函数代码的情况下，给函数添加额外功能。可以把它理解为一种“包装”机制。

装饰器接收一个函数作为参数，并返回一个增强后的函数。语法上用 `@decorator_name` 的形式应用。

装饰器是 Python 中实现切面编程（AOP）的优雅方式，能够有效分离核心逻辑和辅助功能，提高代码的可维护性和复用性。

## 使用示例

1. **基础示例**：
```python
# 定义一个简单的装饰器
def timer(func):
    import time
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} 执行耗时: {time.time() - start:.2f}秒")
        return result
    return wrapper

# 使用装饰器
@timer
def slow_function(duration):
    time.sleep(duration)
    return "完成"

slow_function(2)  # 输出: slow_function 执行耗时: 2.00秒
```

2. **带参数的装饰器**：
```python
def repeat(times):
    """重复执行指定次数"""
    def decorator(func):
        def wrapper(*args, **kwargs):
            results = []
            for _ in range(times):
                results.append(func(*args, **kwargs))
            return results
        return wrapper
    return decorator

@repeat(3)
def greet(name):
    return f"Hello, {name}!"

print(greet("Alice"))  
# 输出: ['Hello, Alice!', 'Hello, Alice!', 'Hello, Alice!']
```

3. **保留元数据**：使用 `functools.wraps` 保留原函数的名称、文档字符串等元数据
```python
import functools

def my_decorator(func):
    @functools.wraps(func)  # 关键！
    def wrapper(*args, **kwargs):
        """装饰器的内部函数"""
        return func(*args, **kwargs)
    return wrapper
```

4. **类作为装饰器**
```python
class CountCalls:
    def __init__(self, func):
        self.func = func
        self.count = 0
    
    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"函数 {self.func.__name__} 已被调用 {self.count} 次")
        return self.func(*args, **kwargs)

@CountCalls
def say_hello():
    print("Hello!")

say_hello()  # 输出: 函数 say_hello 已被调用 1 次 \n Hello!
```

## 执行顺序

装饰器从下往上应用，执行时从上往下：
```python
@decorator_a
@decorator_b
def func():
    pass

# 等价于: func = decorator_a(decorator_b(func))
```

