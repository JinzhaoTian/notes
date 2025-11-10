模板偏特化（Partial Template Specialization）是 C++ 中一种强大的模板技术，它允许你为模板参数的部分特定情况提供特殊实现，而不是完全特化所有参数。

- **主模板**：最通用的模板定义
- **全特化**：为所有模板参数指定具体类型
- **偏特化**：只对部分模板参数或参数特性进行特化

```c++
// 主模板
template <typename T, typename U>
class MyClass {
    // 通用实现
};

// 偏特化：当两个类型相同时
template <typename T>
class MyClass<T, T> {
    // 特殊实现
};

// 偏特化：当第二个参数是int时
template <typename T>
class MyClass<T, int> {
    // 特殊实现
};

// 偏特化：当第一个参数是指针时
template <typename T, typename U>
class MyClass<T*, U> {
    // 特殊实现
};
```

**应用场景**

1. **针对特定类型组合优化**：当某些类型组合有更高效的实现方式时
2. **处理指针类型**：为指针类型提供特殊处理
3. **类型特征匹配**：基于类型特征（如是否为指针、引用等）提供不同实现

#### 函数模板偏特化

C++ 标准不允许函数模板偏特化，但可以通过重载或类模板静态方法实现类似效果：

```c++
// 主函数模板
template <typename T, typename U>
void func(T a, U b) { /*...*/ }

// 不允许的函数模板偏特化
// template <typename T>
// void func<T, int>(T a, int b) { /*...*/ }

// 替代方案：使用类模板静态方法
template <typename T, typename U>
struct FuncImpl {
    static void doFunc(T a, U b) { /*...*/ }
};

template <typename T>
struct FuncImpl<T, int> {
    static void doFunc(T a, int b) { /*...*/ }
};

template <typename T, typename U>
void func(T a, U b) {
    FuncImpl<T, U>::doFunc(a, b);
}
```



#### 偏特化与SFINAE

偏特化常与 SFINAE（Substitution Failure Is Not An Error）技术结合，用于更精细的模板选择：

```c++
template <typename T, typename = void>
struct MyTraits {
    // 默认实现
};

template <typename T>
struct MyTraits<T, std::enable_if_t<std::is_integral_v<T>>> {
    // 仅当T是整型时的特化
};
```