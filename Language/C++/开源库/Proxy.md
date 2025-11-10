[proxy](https://github.com/microsoft/proxy) 是微软开源的一个 C++ 代理模式库，提供了一种现代化的方式来实现动态多态（Dynamic Polymorphism），而无需使用传统的虚函数机制。

## 主要特点

- **非侵入式设计**：不需要修改现有类或继承基类，即可实现多态调用。
- **零虚函数开销**：相比传统虚函数调用，性能可能更优（取决于使用场景）。
- **C++20 支持**：利用现代 C++ 特性（如 Concepts、`std::invoke` 等）。
- **轻量级**：仅需包含单头文件 `proxy.h`，易于集成。


## 核心概念

1. `pro::proxy<T>`
	- 代理类，用于存储和管理目标对象。 
	- 类似于 `std::function`，但支持多方法调用。
2. `pro::dispatch<Ret(Args...)>`
	- 定义调用接口，类似于虚函数声明。
	- 通过 `operator()` 实现具体调用逻辑。
3. `pro::facade<Dispatch...>`
	- 定义代理的接口，类似于抽象基类。
	- 可以组合多个 `dispatch`。

## 示例代码

#### 传统虚函数方式

```cpp
struct Shape {
    virtual ~Shape() {}
    virtual void draw() = 0;
};

struct Circle : Shape {
    void draw() override { std::cout << "Circle\n"; }
};

int main() {
    std::vector<Shape*> shapes;
    shapes.push_back(new Circle());
    shapes[0]->draw();  // 虚函数调用
}
```

#### 使用 `microsoft/proxy`

```cpp
#include "proxy.h"

struct Draw : pro::dispatch<void()> {
    template <class T>
    void operator()(T& self) { self.draw(); }
};

struct Shape : pro::facade<Draw> {};

struct Circle {
    void draw() { std::cout << "Circle\n"; }
};

int main() {
    std::vector<pro::proxy<Shape>> shapes;
    shapes.emplace_back(pro::make_proxy<Shape>(Circle{}));
    shapes[0].invoke<Draw>();  // 代理调用
}
```

**优势**：
- `Circle` 不需要继承 `Shape`，减少耦合。
- 没有虚表（vtable）开销，可能更高效。

