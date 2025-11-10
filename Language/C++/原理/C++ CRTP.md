CRTP（Curiously Recurring Template Pattern，奇异递归模板模式）是一种 C++ 模板编程技术，其**核心特点**是：一个类（派生类）继承自一个以自身（派生类）作为模板参数的类模板（基类），来实现编译期多态。

**为什么用**：
1. **性能**：消除运行时多态（虚函数）的开销。
2. **代码复用**：为一系列相关的类提供通用的实现（Mixin），减少代码重复。
3. **接口强化**：强制派生类实现某些方法（如果基类方法依赖于它们），并在编译期检查。


## 语法结构

```cpp
// 基类是一个模板
template <typename Derived>
class Base {
    // ... 基类的实现，可以使用 Derived 类型
};

// 派生类：继承自以“自己”为模板参数的 Base
class Derived : public Base<Derived> { // 关键就在这里：Base<Derived>
    // ... 派生类的实现
};
```

虽然 `Base<Derived>` 看起来像是基类依赖于一个尚未完全定义的派生类，但在 C++ 中，类的成员函数体是在类定义完成后才被实例化的（除非是内联的）。因此，在 `Base` 类模板中使用的 `Derived` 类型，只有在 `Derived` 类完全定义后才会被检查，这是合法的。


## 主要用途

CRTP 的核心思想是**在编译期实现多态**，从而避免运行期使用虚函数（`virtual`）带来的开销（如虚函数表指针、虚函数表查找等）。它通过静态向下转换（`static_cast`）来实现。

1. **静态多态（编译期多态）**：通过在基类中调用派生类的实现，来实现多态行为，但所有决议都在编译期完成。
```cpp
#include <iostream>
#include <memory>

// CRTP 基类模板
template <typename Derived>
class Cloneable {
public:
    // 注意：返回类型是 std::unique_ptr<Derived>
    std::unique_ptr<Derived> clone() const {
        // 关键：使用 static_cast 将 this 转换为派生类指针
        // 然后调用派生类的拷贝构造函数（必须存在）
        return std::make_unique<Derived>(static_cast<const Derived&>(*this));
    }
};

// 派生类
class ConcreteShape : public Cloneable<ConcreteShape> {
public:
    int width = 100;
    // 必须提供拷贝构造函数，clone() 方法才能工作
    ConcreteShape(const ConcreteShape& other) : width(other.width) {
        std::cout << "ConcreteShape copied!" << std::endl;
    }
    ConcreteShape() = default;
};

int main() {
    ConcreteShape original;
    auto cloned_ptr = original.clone(); // 调用基类的 clone()
    std::cout << “Cloned width: ” << cloned_ptr->width << std::endl; // 输出 100

    return 0;
}
```
- **工作原理：**
	1. `Cloneable<ConcreteShape>` 被实例化。
	2. 它的 `clone()` 方法返回 `std::unique_ptr<ConcreteShape>`。
	3. 在 `clone()` 方法内部，`*this` 被静态转换为 `const ConcreteShape&`，这之所以安全是因为我们确信 `this` 真正指向的是一个 `ConcreteShape` 对象（因为继承关系是 `ConcreteShape : public Cloneable<ConcreteShape>`）。
	4. 然后使用该引用调用 `ConcreteShape` 的拷贝构造函数。

这种方式完全在编译期确定，没有运行时的虚函数开销。


2. **添加通用功能（Mixin）**：CRTP 基类可以为所有派生类注入一些通用的成员函数，这些函数可能依赖于派生类的特定实现。

```cpp
template <typename Derived>
class EqualityComparable {
public:
    // 基类提供 !=，它基于派生类必须提供的 == 来实现
    bool operator!=(const Derived& other) const {
        return !(static_cast<const Derived&>(*this) == other);
    }
};

class MyValue : public EqualityComparable<MyValue> {
public:
    int value;
    MyValue(int v) : value(v) {}

    // 派生类只需要实现 ==
    bool operator==(const MyValue& other) const {
        return value == other.value;
    }
    // != 操作符由基类自动提供
};

int main() {
    MyValue v1(10), v2(20);
    std::cout << (v1 == v2) << std::endl; // 输出 0 (false)
    std::cout << (v1 != v2) << std::endl; // 输出 1 (true)，调用基类的 !=

    return 0;
}
```



## 与动态多态（虚函数）的对比

|特性|CRTP (静态多态)|动态多态 (虚函数)|
|---|---|---|
|**绑定时间**|**编译期**|**运行期**|
|**性能**|**无运行时开销**（无虚表查找）|有轻微开销（虚表指针，间接调用）|
|**灵活性**|类型在编译期确定，不灵活|运行时可通过基类指针操作不同派生类|
|**二进制兼容**|好|可能受虚表布局影响|
|**用途**|性能关键代码，需要避免虚函数开销时|需要运行时灵活处理不同子类时|
