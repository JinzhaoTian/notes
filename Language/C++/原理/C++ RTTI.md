RTTI（运行时类型信息，Run-Time Type Information / Run-Time Type Identification）是 C++ 语言的一项特性，它允许程序**在运行时获取对象的实际类型信息**，这对于面向对象编程中的多态性至关重要。

C++ 的 RTTI 主要提供两个操作符：

1. **`typeid` 操作符**： 返回一个 `std::type_info` 对象的引用，该对象包含类型的名称和比较功能，可以用来查询对象的准确类型。
```cpp
#include <typeinfo>

Base* ptr = new Derived();
if (typeid(*ptr) == typeid(Derived)) {
	// 我们知道 ptr 实际指向一个 Derived 对象
}
```

2. **`dynamic_cast` 操作符**： 用于在继承层级中进行安全的下行转换或交叉转换。如果转换失败（即指针所指对象不是目标类型或其派生类），对于指针类型会返回 `nullptr`，对于引用类型会抛出 `std::bad_cast` 异常。
```cpp
Derived* d_ptr = dynamic_cast<Derived*>(ptr); // 安全转换
if (d_ptr) {
	// 转换成功，可以使用 d_ptr
}
```