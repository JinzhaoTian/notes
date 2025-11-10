`.inl` 文件通常用于存放 [C++](../C++/C++.md) 中的**内联函数**或**模板实现**，虽然 `.inl` 不是标准扩展名，但它常用于将实现代码与声明分离，保持头文件简洁。

### 主要用途

1. **内联函数**：将内联函数的定义放在 `.inl` 文件中，然后在头文件中包含该文件。
2. **模板实现**：模板的实现通常需要放在头文件中，使用 `.inl` 文件可以将实现与声明分离。

### 示例

假设有一个头文件 `example.h` 和一个 `.inl` 文件 `example.inl`：

**example.h**

```cpp
#ifndef EXAMPLE_H
#define EXAMPLE_H

template <typename T>
class Example {
public:
    void doSomething(T value);
};

#include "example.inl"

#endif // EXAMPLE_H
```

**example.inl**

```cpp
template <typename T>
void Example<T>::doSomething(T value) {
    // 实现代码
}
```

### 优点

- **代码组织**：保持头文件简洁，便于维护。
- **可读性**：分离声明和实现，提升代码可读性。

### 注意事项

- **非标准扩展名**：`.inl` 不是 C++ 标准的一部分，使用与否取决于项目规范。
- **编译依赖**：修改 `.inl` 文件可能导致重新编译包含它的源文件。

总结来说，`.inl` 文件用于存放内联函数或模板实现，帮助组织代码并提升可读性。