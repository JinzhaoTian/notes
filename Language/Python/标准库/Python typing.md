Python `typing` 模块是用于类型提示（Type Hints）的标准库，它允许你在代码中添加类型信息，帮助静态类型检查工具（如 `mypy`、`pyright`）在运行前发现类型错误。


## 基本语法

```python
from typing import List, Dict, Optional, Union

# 函数类型注解
def greet(name: str) -> str:
    return f"Hello, {name}"

# 变量类型注解
age: int = 25
names: List[str] = ["Alice", "Bob"]

# 复杂类型
def process_items(items: List[int], config: Dict[str, str]) -> Optional[str]:
    if items:
        return str(items[0])
    return None

# 联合类型（Python 3.10+ 可用 | 语法）
def parse_value(value: Union[int, str]) -> int:  # 旧写法
    return int(value)

def parse_value(value: int | str) -> int:  # 新写法（3.10+）
    return int(value)
```


## 常用类型

1. `List[T]`：列表，元素类型为 `T`
2. `Dict[K, V]`：字典，键类型 `K`，值类型 `V`
3. `Optional[T]`：可为 `T` 或 `None`
4. `Union[T1, T2]`：多种类型之一
5. `Tuple[T1, T2]`：元组
6. `Any`：任意类型（绕过检查）


## 高级特性

```python
from typing import TypeVar, Callable, Protocol

# 泛型
T = TypeVar('T')
def first(items: List[T]) -> T:
    return items[0]

# 可调用对象
def apply(func: Callable[[int, int], int], x: int, y: int) -> int:
    return func(x, y)

# 协议（结构化子类型）
class Drawable(Protocol):
    def draw(self) -> None: ...

def render(obj: Drawable) -> None:
    obj.draw()
```