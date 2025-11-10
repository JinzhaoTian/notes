
### SFINAE

SFINAE（Substitution Failure Is Not An Error，替换失败并非错误）是 C++ 模板元编程中的一个重要概念，它是 C++ 模板重载决议过程中的一条规则。

> [!基本概念]
> 当编译器在**模板实例化**过程中遇到无效的代码时：如果这种无效性是由于尝试用某个类型替换模板参数导致的，那么**编译器不会报错**，而是**简单地忽略这个候选模板**，继续寻找其他可行的模板。

SFINAE 是 C++ 模板元编程的强大工具，虽然现代 C++ 提供了更简洁的替代方案（如概念），但理解 SFINAE 对于处理遗留代码和深入理解模板机制仍然很重要。

**工作原理**

在模板重载解析过程中：
- 编译器会尝试用实际类型替换模板参数
- 如果替换导致无效代码（如访问不存在的成员、无效的类型操作等）
- 这个模板候选会被静默丢弃，而不是产生编译错误
- 只要还有其他有效候选，编译就会继续

#### 常见用途

1. **类型特征检查**：检查类型是否具有特定成员或支持特定操作
```c++
template <typename T, typename = void>
struct has_foo : std::false_type {};

template <typename T>
struct has_foo<T, std::void_t<decltype(std::declval<T>().foo())>> : std::true_type {};
```

2. **函数重载控制**：基于类型特性启用/禁用特定重载
```c++
template <typename T>
typename std::enable_if<std::is_integral<T>::value, void>::type
process(T value) { /* 处理整数 */ }

template <typename T>
typename std::enable_if<std::is_floating_point<T>::value, void>::type
process(T value) { /* 处理浮点数 */ }
```

3. **模板特化选择**：根据类型属性选择不同的实现

#### C++11/14/17 的改进

- `std::enable_if`: 经典的 SFINAE 工具
- `std::void_t`: 简化类型特征检查
- `std::declval`: 在不构造对象的情况下获取类型的右值引用
- （C++17）`if constexpr`: 可以替代部分 SFINAE 的使用场景

#### 示例

```c++
#include <iostream>
#include <type_traits>

// 只有T有reserve()成员函数时才启用这个重载
template <typename T>
auto foo(T& container) -> decltype(container.reserve(0), void()) {
    std::cout << "with reserve\n";
    container.reserve(10);
}

// 后备重载
template <typename T>
void foo(T& container) {
    std::cout << "without reserve\n";
}

int main() {
    std::vector<int> v;  // 有reserve()
    foo(v);              // 调用第一个重载
    
    std::list<int> l;    // 无reserve()
    foo(l);              // 调用第二个重载
}
```


#### 现代 C++ 替换方案

1. （C++20）Concepts：**最直接的替代方案**，提供了更清晰、更易读的语法来表达模板约束。
2. （C++17）`if constexpr`：允许在编译时进行条件判断，可以替代部分 SFINAE 的使用场景。
3. （C++20）Requires 表达式：可以与 concepts 结合使用，提供更灵活的约束表达方式。


### Concepts

C++ Concepts 是 C++20 引入的一项重要特性，它是对模板编程的重大改进，用于指定模板参数必须满足的要求。

> [!基本概念]
Concepts 是一种**编译时谓词**（返回布尔值的表达式），用于约束模板参数，使得编译器能够:
> - 更早地检测模板使用错误
> - 生成更清晰的错误信息
> - 提供更好的代码可读性

**基本语法**

```c++
// 定义一个概念
template<typename T>
concept MyConcept = requires(T a, T b) {
    { a + b } -> std::convertible_to<T>;
    { a == b } -> std::same_as<bool>;
};

// 使用概念约束模板
template<MyConcept T>
void myFunction(T value) { /*...*/ }
```

**主要特点**

1. **类型约束**：指定模板参数必须满足的接口要求
2. **编译时检查**：在编译时验证类型是否符合要求
3. **更好的错误信息**：当类型不满足要求时，错误信息更清晰
4. **函数重载**：可以根据概念不同进行函数重载

#### 常用概念

C++20 标准库提供了许多预定义的概念，例如：
- `std::integral` - 要求类型是整数类型
- `std::floating_point` - 要求类型是浮点类型
- `std::same_as` - 要求类型完全相同
- `std::convertible_to` - 要求类型可转换
- `std::invocable` - 要求类型可调用

#### 示例

```c++
#include <concepts>
#include <iostream>

// 定义一个要求类型可加的概念
template<typename T>
concept Addable = requires(T a, T b) {
    { a + b } -> std::same_as<T>;
};

// 使用概念约束的函数
template<Addable T>
T add(T a, T b) {
    return a + b;
}

// 另一个例子：要求类型有size()方法
template<typename T>
concept HasSize = requires(T t) {
    { t.size() } -> std::integral;
};

template<HasSize T>
void printSize(const T& container) {
    std::cout << "Size: " << container.size() << "\n";
}

int main() {
    std::cout << add(3, 4) << "\n";    // 正确
    // std::cout << add("a", "b");    // 错误: 字符串字面量不满足Addable
    
    std::vector<int> v{1, 2, 3};
    printSize(v);  // 正确
}
```