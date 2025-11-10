

## 可变参数模板

```C++
template <typename T, typename... Args>
void foo(const T &t, const Args& ... rest);
```

1. **递归展开**：最常见的用法是通过递归方式处理参数包
```cpp
// 基础情况
void print() {
    std::cout << std::endl;
}

// 递归情况
template<typename T, typename... Args>
void print(T first, Args... args) {
    std::cout << first << " ";
    print(args...); // 递归调用
}
```

2. **折叠表达式**（C++17）：C++17 引入了折叠表达式，简化了可变参数模板的使用
```cpp
template<typename... Args>
auto sum(Args... args) {
    return (args + ...); // 折叠表达式
}
```

3. **完美转发**
```cpp
template<typename... Args>
void wrapper(Args&&... args) {
    some_function(std::forward<Args>(args)...);
}
```

4. **计算参数数量**
```cpp
template<typename... Args>
constexpr std::size_t countArgs(Args...) {
    return sizeof...(Args); // 使用 sizeof... 获取参数包大小
}
```

5. **元组实现**：可变参数模板是实现 `std::tuple` 的基础：
```cpp
template<typename... Types>
class Tuple;

template<typename Head, typename... Tail>
class Tuple<Head, Tail...> : private Tuple<Tail...> {
    Head head;
    // ...
};

template<>
class Tuple<> {}; // 终止条件
```
