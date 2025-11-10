移动语义（Move Semantics）是 C++ 11 引入的一项重要特性，它允许资源（如动态内存、文件句柄等）的所有权从一个对象转移到另一个对象，而不是进行昂贵的复制操作。

传统 C++ 中，当对象包含动态分配的资源时，复制操作可能很昂贵：
```c++
std::vector<int> createLargeVector();  // 返回一个大vector
std::vector<int> v = createLargeVector();  // 传统C++会进行深拷贝
```

移动语义允许移动而非复制这些资源，避免不必要的深拷贝，直接窃取右值的资源（如动态内存、文件句柄），提高性能，
```c++
std::string s1 = "Hello";
std::string s2 = std::move(s1);  // 移动构造，s1 变为空
```

其中 `std::move` 只是将左值强制转换为右值引用，表示可以移动该对象，并不真正移动数据。

## 核心概念

1. **右值引用**
	- 使用 `&&` 表示，如 `T&&`，它可以绑定到临时对象（右值）
2. **移动构造函数**
3. **移动赋值运算符**



## 使用示例

```c++
class MyString {
public:
    // 移动构造函数
    MyString(MyString&& other) noexcept : 
	    data(other.data),
	    size(other.size)
	{
        other.data = nullptr;  // 使原对象处于有效但空的状态
        other.size = 0;
    }
    
    // 移动赋值运算符
    MyString& operator=(MyString&& other) noexcept
    {
        if (this != &other) {
            delete[] data;  // 释放当前资源
            data = other.data;
            size = other.size;
            other.data = nullptr;
            other.size = 0;
        }
        return *this;
    }
    
private:
    char* data;
    size_t size;
};
```

## 触发场景

在 C++ 中会在以下情况下自动触发或可以手动触发移动语义，

1. **从右值构造或赋值**：
```c++
std::vector<int> getVector()
{
	return {1, 2, 3};
}

std::vector<int> v = getVector();   // 自动触发移动语义（如果可用）
```

2. **显式使用 `std::move`**：
```c++
std::string str1 = "Hello";
std::string str2 = std::move(str1); // 显式触发移动语义
```

3. **函数返回局部对象**（C++ 17 起触发 [NRVO](../构建/NRVO.md) 优化）：
```c++
std::vector<int> createVector() {
    std::vector<int> vec;
    // ...填充数据...
    return vec;                     // 可能触发移动语义或 NRVO（返回值优化）
}
```

4. **标准库容器操作**：
```c++
std::vector<std::string> vec;
std::string s = "example";
vec.push_back(std::move(s));        // 移动而非复制
```

5. **交换操作**（`swap`）
```c++
std::string a = "foo", b = "bar";
std::swap(a, b);                    // 内部使用移动语义
```

6. **构造/赋值时使用右值**
```c++
class MyClass {
public:
    MyClass(MyClass&& other); // 移动构造函数
    MyClass& operator=(MyClass&& other); // 移动赋值运算符
};

MyClass obj1;
MyClass obj2 = std::move(obj1); // 触发移动构造
```

7. **标准库算法**
```c++
std::vector<std::string> vec1, vec2;
// ...填充vec1...
std::move(vec1.begin(), vec1.end(), std::back_inserter(vec2)); // 移动元素
```

8. **完美转发**
```c++
template<typename T>
void wrapper(T&& arg) {
    func(std::forward<T>(arg)); // 可能触发移动语义
}
```


## 不触发情况

1. 对象没有移动构造函数/移动赋值运算符
2. 对象被声明为 const（无法移动）
3. 基本数据类型（int, float 等）的移动等同于复制

## 注意事项

1. 移动后源对象处于有效但未指定状态
2. 移动操作通常应标记为 `noexcept`
3. 对于具有资源管理责任的类，应实现移动语义以获得最佳性能