应该总结每个语言的语法情况，包括数据类型，字符串特性，内存特性，常用的库，刷题时常用的特点等等，不然每次都会忘了好多。


## 变量和类型

### 基本内置类型

关键字为 bool，char，short，int，long，long long，float，double，long double

unsigned short，unsigned int，unsigned long，unsigned long long，

### 复合类型

复合类型是指基于其他类型定义的类型。

#### 引用

引用（reference）为对象起了另外一个名字，引用和它的初始值对象一直绑定在一起，必须初始化。在C++语言中，通常所说的引用，其实指的是“左值引用”。

左值指既能够出现在等号左边，也能出现在等号右边的变量；右值则是**只能出现在等号右边**的变量。
- 左值是一个表示数据的表达式，如变量名或解除引用的指针，程序可以获得其地址。
- 右值一般是不可寻址的常量，或在表达式求值过程中创建的无名临时对象，短暂性的，如字面常量，表达式x+y等，以及有返回值的函数。
左值有持久的状态，而右值要么是字面常量，要么是在表达式求值过程中创建的临时变量。

##### 左值引用
```C++
int i = 9;
int &j = i; 
```


##### 右值引用

传统的C++引用（现在说是左值引用）可以用 `&` 标志符，右值引用可以用 `&&` 标志符。右值引用可以关联到右值，即可以出现在赋值表达式右边，但不能对其应用地址运算符的值。**引入右值引用的主要目的之一是实现移动语义**。

```C++
int i = 42;
int &r = i;        // 正确，r引用i
int &&rr = i;      // 错误，i是一个左值
int &r = i * 4;    // 错误，i * 4 是一个右值
int &&rr = i * 4;  // 正确
```

**移动语义**：将临时数据的所有权转移到另一个变量，避免重复的开辟空间复制数据等一系列操作。

比如，在某些类成员函数中声明的一个对象，这个对象的生存周期是在这个成员函数中，
```c++
Obj ObjFactory::CreateObj() {
    Obj localObj;
    
	// ...
    
    return localObj;
}
```

此时在外面某处调用该方法时，
```C++
Obj outObj = factory.CreateObj();
```
该处会发生一个拷贝复制，调用拷贝复制构造函数，将内部的 localObj 对象 “传” 出来。但是此时对象 localObj 不是对象 outObj，outObj 是一个新造的对象。并且，该方法调用结束时，会调用内部对象 localObj 的析构函数，也就是说总共发生了一次拷贝一次释放，会耗费时间不说，如果对象内部有一些管理的资源，析构函数的调用会造成资源的误释放。

此时为了提高效率，可以实现借助右值和移动语义，将 localObj 真正的 “传” 给 outObj，
```c++
class Obj {
public:
    Obj();
    ~Obj();
    
    Obj(Obj&& other) noexcept :
		value(other.value) {
		    other.value = 0;
	}

    Obj& operator=(Obj&& other) noexcept{
	    if (this != &other) {
	        if (value != 0) {
	            // 释放资源
	        }
	
	        value = other.value;
	        other.value = 0;
	    }
	    return *this;
	}

private:
	int value;

    Obj(const Obj&) = delete;
    Obj& operator=(const Obj&) = delete;
};
```

此时，ObjFactory 的成员函数可以改造为，
```c++
Obj ObjFactory::CreateObj() {
    Obj localObj;
    
	// ...
    
    return std::move(localObj);
}
```

在外层调用中，
```c++
Obj outObj = std::move(factory.CreateObj());
```
这样全程不会发生拷贝和析构的调用，但是移动后 localObj 处于空状态。






#### 指针

引用是变量的别名，**指针就是变量地址的别名。** 与引用类似，指针也实现了对其他对象的间接访问。然而指针与引用又有很多不同点：
1. **指针本身是一个对象**，允许对指针赋值和拷贝。而且在指针的声明周期内它可以先后指向几个不同的对象。
2. 指针无须在定义时赋初始值。（不太建议这个做法）和其他内置类型一样，在块作用域内定义的指针如果没有被初始化，也将拥有一个不确定的值。

```C++
int i = 42;
int *p1 = 0;          // 等同于int *p1=nullptr;
int *p2 = &i;
int *p3;              // 不推荐
```

过去的程序还会用到一个名为 `NULL` 的预处理变量（preprocessor variable）来给指针赋值，这个变量在头文件 `cstdlib` 中定义，它的值为0。在新标准下，**现在的C++程序最好使用 `nullptr` ，同时尽量避免使用 `NULL` .**


##### 智能指针

智能指针是一个类，当超出了类的作用范围时，类会自动调用析构函数，自动释放资源。智能指针利用这个原理来自动化的释放内存空间，避免内存泄漏。
 
1. auto_ptr：C++98标准，C++11抛弃
2. unique_ptr：一个对象只能有一个unique_ptr指向它
3. shared_ptr：一个对象可以有多个 shared_ptr 指向它，会有引用计数来统计。
4. weak_ptr：搭配shared_ptr 使用，避免死锁。


### 结构体

```C++
struct TreeNode {
	int val;
	TreeNode *left, *right;
	TreeNode() : val(0), left(nullptr), right(nullptr) {}
};                              // 别忘了分号
```


**类和结构体**：它们之间唯一的区别就是，结构体默认访问类型是public，而类是private。


### const

- const指针：将 `*` 放在const之前表示指针是一个常量，即不变的是指针本身的值而非指向的那个值。
- 








## 命名空间

C++语言引入命名空间（Namespace），主要是为了**避免命名冲突**，其关键字为 namespace。

```C++
namespace hello {
	int val = 1;
}

using namespace hello;   // 引入命名空间，尽量不要使用，防止污染命名空间
using hello::val;        // 使用using声明
```


## 表达式和语句


### 强制转换

1. 隐式强制转换

2. 显式强制转换
	- `static_cast` ：任何具有明确定义的类型转换，只要不包含底层 const 。
	- `const_cast` ：只能改变运算对象的底层 const ，也就是说能将对象的 const 属性去除掉，但是不能改变对象的类型。
	- `reinterpret_cast` ：为运算对象的位模式提供较低层次上的重新解释，如将一个int型变量指针转化成一个char型变量指针，意味着所占字节数以及变量数量会发生变化。



## 函数

### 参数传递

引用传递和值传递



### 内联函数

函数调用一般比求等价表达式的值更慢一点，将函数指定为内联函数，那么在编译时，编译器会把该函数的代码副本放置在每个调用该函数的地方。

```C++
// 声明
inline const string &shorterString(const string &s1, const string &s2) {
	return s1.size() <= s2.size() ? s1 : s2;
}
```

**在编译过程中**，就会将 `cout << shorterString(s1, s2) << endl;` 展开为 `cout << (s1.size() <= s2.size() ? s1 : s2) << endl;`

内联函数一般用于优化规模小、流程直接、调用频繁的函数。

###  函数指针

和变量一样，函数在内存中有固定的地址。函数的实质也是内存中一块固定的空间。

```C++
int foo(int x) {
    return x;
}

int add(int a, int b) {
    return a + b;
}

int sub(int a, int b) {
    return a - b;
}

void func(int e, int d, int(*f)(int a, int b)) {
    // 传入了一个int型，双参数，返回值为int的函数
    std::cout<<f(e, d)<<std::endl;
}

int main() {
    int (*funcPtr)(int) = foo; 
    (*funcPtr)(5);                 // 通过funcPtr调用foo(5)
    funcPtr(5)                     // 也可以这么使用，在一些古老的编译器上可能不行
    
    func(2, 3, add);
    func(2, 3, sub);
    
    return 0;
}
```


## 类

```C++
class Sample_class {
    friend std::ostream& print(std::ostream&, const Sample_class&);     // 友元函数：允许访问非公有成员
public:
    // 构造函数
    Sample_class() = default;                                           // 默认构造函数：使用 = default 来要求编译器生成构造函数
                                                                        //             只有类没有任何构造函数时，才会生成默认构造函数
    Sample_class(const std::string& s, int n) : name(s), num(n) {}      
    Sample_class(const std::string& s) : name(s), num(0) {}
    Sample_class(std::string s) : Sample_class(s, 0) {}                 // 委托构造函数
    Sample_class(const Sample_class&);                                  // 拷贝构造函数


    // 公有成员函数
    std::string getName() const { return name; }                        // 定义在类内部的函数是隐式的inline函数
    Sample_class &combine(const Sample_class&);                         // 成员函数的声明

    static int getTotalNum() { return total_num; }

private:
    // 私有成员变量
    std::string name;
    int num;

    static int total_num;                                               // 静态成员函数：只保存一份

    // 私有成员函数
    int getNum() const { return num; }
};

Sample_class::Sample_class(const Sample_class &orig): name(orig.name), num(orig.num) { }

Sample_class& Sample_class::combine(const Sample_class& rhs) {
    num += rhs.num;
    return *this;
}

std::ostream& print(std::ostream& os, const Sample_class& item) {
    os << item.getName() << ' ' << item.getNum() << std::endl;
    return os;
}
```


### 实例化

1. 隐式创建
```c++
Employee empOne;       // 调用无参构造函数
Employee empTwo(2);    // 调用有参构造函数
```


2. 显示创建：在进程栈空间进行分配内存，其分配和释放由系统决定。
```c++
Employee empOne = Employee();            // 调用无参构造函数
Employee empTwo = Employee(2);           // 调用有参构造函数
```

3. new 创建：在堆上分配内存，这部分内存由程序员自己管理，需要显式调用 free 进行释放。
```c++
Employee *empOne = new Employee();       // 调用无参构造函数
Employee *empTwo = new Employee(2);      // 调用有参构造函数
```


### [静态变量](C++%20静态变量.md)

