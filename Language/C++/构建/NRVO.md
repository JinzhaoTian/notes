NRVO（Named Return Value Optimization，命名返回值优化）是 C++ 编译器中的一种优化技术，用于消除函数返回局部对象时的额外拷贝或移动操作。

## 基本概念

当函数返回一个局部对象时，如果没有优化，通常会发生以下步骤：

1. 在函数内部构造局部对象
2. 将局部对象拷贝或移动到一个临时对象中（返回值）
3. 将临时对象拷贝或移动给调用方的接收对象

NRVO 允许编译器跳过中间的拷贝/移动步骤，直接在调用方准备的内存位置上构造返回的对象。


## 使用示例

```c++
std::string createString() {
    std::string s = "Hello, NRVO!";
    return s;  // 通常这里会触发 NRVO
}

int main() {
    std::string result = createString();  // s直接在result的内存位置构造
}
```


## RVO

RVO（Return Value Optimization，返回值优化）适用于返回匿名临时对象，
```c++
std::string createString() {
    return std::string("Hello, RVO!");  // 触发 RVO
}
```

NRVO 适用于返回命名局部对象（Named Return Value Optimization）。


## 注意事项

1. NRVO 不是强制要求的，但大多数现代编译器都会实施
2. 在某些复杂情况下（如多个返回路径返回不同对象），NRVO 可能无法应用
3. **C++ 17 开始，返回值优化在某些情况下成为强制要求**（当返回与函数返回类型相同的局部变量时）
4. NRVO 能显著提高返回大型对象的效率，是 C++ 性能优化中的重要技术之一

