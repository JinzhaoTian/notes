Lambda 表达式是 C++ 11 引入的一种匿名函数特性，它允许你在需要函数的地方内联定义函数，而无需单独声明和定义命名函数。

## 基本语法

引入 Lambda 表达式的前导符是一对方括号 `[]`，称为 Lambda 引入符（lambda-introducer），Lambda 表达式的返回值类型是语言自动推断的。

```c++
[capture list](parameter list) -> return type { function body }
```
其中：
- `capture list`：定义 lambda 表达式可以从外部作用域捕获哪些变量
- `parameter list`：与普通函数的参数列表类似
- `return type`：可选的，如果省略则由编译器自动推导
- `function body`：包含 lambda 表达式的代码

如，
```C++
[](float a, float b) -> bool { return std::abs(a) < std::abs(b); }
```


## 捕获列表

Lambda 可以通过捕获列表访问外部变量：
1. **值捕获**：
	- `[x]`：捕获变量的副本
2. **引用捕获**：
	- `[&x]`：捕获变量的引用
3. **隐式捕获**：
    - `[]`：不捕获任何外部变量
    - `[=]`：以值方式捕获所有外部变量
    - `[&]`：以引用方式捕获所有外部变量
4. **混合捕获**：
	- `[x, &y]`：可以组合使用

## 可变 lambda

默认情况下，值捕获的变量在 lambda 内是常量。使用 `mutable` 关键字可以修改这些副本：
```c++
int x = 10;
auto counter = [x]() mutable { 
    x++; 
    return x; 
};
std::cout << counter();  // 输出: 11
std::cout << x;         // 输出: 10 (原始值未改变)
```

## 带返回类型的 lambda

当 lambda 体包含多个返回语句时，可能需要显式指定返回类型：
```c++
auto divide = [](double x, double y) -> double {
    if (y == 0.0) return 0.0;
    return x / y;
};
```

## 常见用途

1. **STL 算法**：
```c++
std::vector<int> v = {1, 2, 3, 4, 5};
std::for_each(v.begin(), v.end(), [](int x) {
	std::cout << x << " ";
});
```

2. **回调函数**：
```c++
button.onClick([]() {
	std::cout << "Button clicked!";
});
```

3. **线程初始化**：
```c++
std::thread t([](){
	std::cout << "Hello from thread!";
});
```

