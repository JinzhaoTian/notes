
## 提前退出

1. **提前退出**：
```cpp
void handle_request(Request req) {
	auto* ctx = new Context();
	if(!auth(req))return; // 提前返回,ctx 水远不会被释放
	process(ctx, req);
	delete ctx; 
}
```
2. **异常路径**：
```cpp
void init() {
	resource_a = malloc(1024);
	risky_call();// 异常, resource_a 泄漏
	resource_b = new char[512];
}
```


### 解决方案

永远不要手动管理资源，换成 RAIl，问题自动消失:
```cpp
void handle_request(Request req) {
	auto ctx = std::make_unique<Context>();
	if(!auth(req)) return; // 自动析构
	process(ctx.get(), req);
}
```

## 释放方式不匹配

`new` 与 `delete`，`new[]` 与 `delete[]]`，`malloc` 与 `free`，这些必须匹配,而且要成对出现。
一旦混用，轻则出现未定义行为，重则堆结构损坏，程序崩溃。
但更隐蔽的是跨模块分配释放。在Windows上，如果你的DLLA用new分配内存，DLLB用delete释放，而两者都静态链接CRT，那么它们各自拥有独立的堆。A在堆1上分配，B却试图在堆2上释放，结果不是泄漏就是crash。
即使你用了动态链接CRT，我也强烈建议你分配和释放必须在同一模块内完成。


## 指针丢失

```cpp
int* p = new int(42);
p=get_next();//原始地址彻底丢失
```

这段代码，看起来犯了低级错误，但在状态机、回调链、或指针数组管理中，这种「覆盖即遗忘」极其常见。


### 解决方案

用 `unique_ptr`，赋值时自动析构旧对象，地址永远不会裸奔。


## 异常安全

在构造函数里分配多块资源，中途抛异常，这是经典的陷阱，前面已分配的资源无法回收。

```cpp
class BadManager {
	char* buf1;
	char* buf2;
public:
	BadManager() {
		buf1 = new char[100]; // 成
		throw std::runtime_error("fail");// buf1 永远不会释放
		buf2 = new char[200];
	}
	
		~BadManager() { 
		delete[] buf1; 
		delete[] buf2;
	}
};
```
这里的析构函数永不会被调用，因为对象构造还没完成就异常了。
解决方案是所有成员都用 RAII 类型，编译器会确保已构造成员按逆序析构。

```cpp
class GoodManager {
	std::unique_ptr<char[]> buf1;
	std::unique_ptr<char[]> buf2;
public:
	GoodManager() {
		buf1 = std::make_unique<char[]>(100);
		throw std::runtime_error("fail");// buf1 自动析构
		buf2 = std::make_unique<char[]>(200);
	}
};
```

## 容器裸指针

```cpp
std::vector<MyObj*> list;
list.push_back(new Myobj());
list.clear();//指针被消空,Myobj实例永存堆上
```

`std::vector` 只管理指针本身，不管理指针指向的对象。
更糟的是部分清理:只删了一半，剩下的忘了。
### 解决方案

容器里放值，或放智能指针:
```cpp
std::vector<std::unique_ptr<MyObj>> list;
list.push_back(std::make_unique<MyObj>());
list.clear();//所有对象自动释放
```

如果接口强制要求裸指针，至少用注释标明所有权:调用方负责释放，并封装成 RAllwrapper。


## Rule of Five

自定义类管理堆内存时，如果只写了析构函数，没处理拷贝/移动语义，就会引发双重释放或泄漏。
```cpp
class String {
	char* data;
public:
	String(const char* s) { data = new char[strlen(s)+1]; strcpy(data, s); }
	~String() { delete[] data; }
	// 缺少拷贝构造-浅拷贝-双重delete
};
```

现代C++里，如果你的成员全是RAll类型，编译器会自动生成正确的移动语义，你甚至不需要写析构函数。

## 非虚析构
 
```cpp
class Base { public: ~Base() {} };

class Derived : public Base {
	char* extra = new char[100];
public:
	~Derived() { delete[] extra; }
};

Base* p = new Derived();
delete p; //只调用Base::~Base(),Derived 部分既不析构,内存也不完整释放
```

这不仅是派生类资源泄漏，更是未定义行为，如堆管理器只释放sizeof(Base)字节，而Derived实际占更多，多出来的字节永远卡在堆里，形成内存碎片。

**所以只要类可能被继承，析构函数就必须为 virtual**。


## `shared_ptr` 循环引用

```cpp
struct Node {
	std::shared_ptr<Node> parent;
	std::vector<std::shared_ptr<Node>> children;
};
auto a = std::make_shared<Node>();
auto b = std::make_shared<Node>();
a->children.push_back(b);
b->parent=a;//循环形成，引用计数水不归零
```

因为对象还在被引用，ASan 和 Valgrind 检测不出来这种泄漏。
它就像慢性毒药那样，Private Dirty 内存每天涨一些，时间长了内存就溢出了。


### 解法方案

用 weak_ptr，但要注意的是并非所有反向引用都要weak_ptr，如果 parent 生命周期明确长于child,shared_ptr 仍是安全的。

## 自定义deleter用错

```cpp
void* raw = malloc(100);
auto ptr = std::shared_ptr<void>(raw); // 默认用 delete, 但 raw 是 malloc 来的
```

结果就会出现未定义行为。

正确的写法是：
```cpp
auto ptr = std::shared_ptr<void>(raw, free);
```

同理，COM 对象、CUDA 显存、mmap 区域，都必须配对正确的释放函数。
智能指针不是万能的，deleter必须显式指定。


## 第三方 API 约定误解

很多C库函数返回的指针需要调用方释放，比如strdup返回的字符串必须free;而sqlite3_column_text返回的指针由sqlite3_finalize管理。搞混了，就是泄漏。
COM更是典型:每次AddRef必须对应一次Release，少一次就等是对象泄漏。

应对策略是用RAII封装所有外部资源：
```cpp
template<typename T> 
class ComPtr {
	T* P_;
public:
	explicit ComPtr(T*p = nullptr):p_(p) {}
	~ComPtr() { if (p_) P_->Release();} // 禁用拷贝，或实现引用转移
};
```