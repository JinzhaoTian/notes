完美转发（Perfect Forwarding）是 C++ 11 引入的一项重要特性，它允许函数模板将其参数**无损地**转发给其他函数，保持参数的原始类型（包括左值/右值属性、const/volatile 限定符等），避免额外的拷贝。

## 基本语法

```c++
template <typename T>
void wrapper(T&& arg) {            // 通用引用（Universal Reference）
    target(std::forward<T>(arg));  // 完美转发
}
```

## 核心概念

完美转发解决了以下问题：

1. 保持参数的值类别（左值还是右值）
2. 保持参数的常量性（const/volatile）
3. 避免不必要的拷贝

## 关键技术

完美转发依赖于两个关键特性：

1. **右值引用（`T&&`）**：可以绑定到左值和右值
2. **`std::forward`** ：有条件地将参数转换为右值


## 工作原理

1. 当传入左值时，`T` 被推导为左值引用类型，`std::forward` 返回左值
2. 当传入右值时，`T` 被推导为非引用类型，`std::forward` 返回右值