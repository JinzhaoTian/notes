C++ 函数对象（Function Object，也叫“仿函数”或“函数符”）是 STL 的一个基本组件，本质是重载了 `operator()` 运算符的类 / 结构体实例，函数对象可以像调用普通函数一样调用它，并且可以有参数和返回值。

## 引入原因

1. **状态保持**：让函数拥有“记忆”
```cpp
#include <algorithm>
#include <vector>
#include <iostream>

class CountEven {
private:
    int count = 0; // 内置计数器，保存状态
public:
    // 每次调用时检查是否为偶数，计数+1
    void operator() (int x) {
        if (x % 2 == 0)
            count++;
    }
    // 提供接口获取计数结果
    int get_count() const {
        return count;
    }
};

int main() {
    std::vector<int> nums = {1,2,3,4,5,6,7,8};
    CountEven counter;
    
    // 把函数对象传给count_if的“变种”for_each
    std::for_each(nums.begin(), nums.end(), counter);
    std::cout << "偶数个数：" << counter.get_count() << std::endl; // 输出4
    
    return 0;
}
```

2. **性能更优**：支持编译期内联
	- 函数指针是间接调用（运行时才确定调用哪个函数），编译器很难优化；而函数对象的 `operator()` 是成员函数，编译器知道具体类型，能直接内联（Inline），减少函数调用开销。
3. **更好的适配性**：STL 算法的 “最佳搭档”
	- STL 算法（如 `sort`、`find_if`）的模板参数需要可调用对象，函数对象的类型唯一性（每个函数对象类是不同类型）让 STL 能做更灵活的适配。
```cpp
#include <functional> // 包含std::not_fn（C++17）

int main() {
    std::vector<int> nums = {1,2,3,4,5};
    CountEven even_counter;
    // 用std::not_fn包装，统计“非偶数”（奇数）
    auto odd_counter = std::not_fn(even_counter);
    // 注意：这里要传引用，否则会拷贝
    std::for_each(nums.begin(), nums.end(), std::ref(odd_counter));
    std::cout << "奇数个数：" << odd_counter.get_count() << std::endl; // 输出3   
    return 0;
}
```


## 分类

函数对象按参数个数的数量可分为三类：
1. 生成器（generator）：无参数，一般很少用
2. 一元函数对象：`operator()` 接受 1 个参数（如前面的 `AddFix`、`IsEven`）
3. 二元函数对象：`operator()` 接受 2 个参数（如前面的 `SortBy`）

 C++ 标准库中的 `std::plus`（实现加法）也是一个二元函数对象：
 ```cpp
#include <functional> // 包含std::plus（C++11+）
int main() {
    std::vector<int> a = {1,2,3};
    std::vector<int> b = {4,5,6};
    std::vector<int> c(3);
    // std::plus是二元函数对象，计算a[i] + b[i]
    std::transform(a.begin(), a.end(), b.begin(), c.begin(), std::plus<int>());
    for (int x : c)
        std::cout << x << " "; // 输出5 7 9
    return 0;
}
 ```

## 与函数指针的差异对比

