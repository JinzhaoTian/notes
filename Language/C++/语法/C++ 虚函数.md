
C++ 中的虚函数的作用主要是实现**运行时多态**。在基类中声明一个虚(virtual)函数，然后在派生类中对其进行重写，基类的引用或者指针指向一个派生类对象，当该基类变量调用该函数时候，会自动调用派生类的函数，这就是所谓的动态多态。

虚函数通常通过**虚函数表**（Virtual Table，vtbl）来实现，在虚表中存储函数指针，实际调用时需要间接访问，因此需要多一点时间花费。

然而，虚函数速度慢的主要原因是编译器在编译时通常并不知道它将要调用哪个函数，因此它**不能被内联优化和其它很多优化**，因此可能会增加很多无意义的指令（准备寄存器、调用函数、保存状态等），而且如果虚函数有很多实现方法，那分支预测的成功率也会降低很多，分支预测错误也会导致程序性能下降。

所以虚函数其实最主要的性能开销在于它阻碍了编译器内联函数和各种函数级别的优化，导致性能开销较大。 

**使用 C++ 本来就是为了性能，不然为什么不用 Python ？**


## Proxy

微软基于 C++20 开发了 [Proxy](https://github.com/microsoft/proxy) 库，使得 C++ 可以很可靠的零成本抽象来表示多态。

- **传统虚函数方式实现动态多态** ：
```c++
#include<iostream>
 
class Model {
public: 
  virtual void show() {
    std::cout << "model run!\n";
  }
};
 
class Boy :public Model{
public:
  void show() override {
    std::cout << "boy run!\n";
  }
};
 
class Girl :public Model{
public:
  void show() override {
    std::cout << "girl run!\n";
  }
};
 
class Man :public Model{
public:
  void show() override {
    std::cout << "man run!\n";
  }
};
 
void justrun(Model & m) {
  m.show();
}
 
int main() {
  Model who;
  Girl girl;
  Boy boy;
  Man man;
 
  justrun(who);
  justrun(girl);
  justrun(boy);
  justrun(man);
}
```


- **使用模版方式实现静态多态** ：
```c++
#include<iostream>
 
template<typename T>
class Model {
public:
  void show() {
    T* p = static_cast<T*>(this);
    p->runit();
  }
  
  void runit() {
     std::cout << "model run!\n";
  }
};
 
class Who :public Model<Who> {};
 
class Boy :public Model<Boy> {
public:
  void runit() {
    std::cout << "boy run!\n";
  }
};
 
class Girl :public Model<Girl> {
public:
  void runit() {
    std::cout << "girl run!\n";
  }
};
 
class Man :public Model<Man> {
public:
  void runit() {
    std::cout << "man run!\n";
  }
};
 
template<typename T>
void justrun(Model<T>& m) {
  m.show();
}

int main() {
  Who who;
  Girl girl;
  Boy boy;
  Man man;
 
  justrun(who);
  justrun(girl);
  justrun(boy);
  justrun(man);
}
```


- 基于 Proxy 实现多态
```c++
#include<iostream>
#include "proxy.h"
 
// 声明一个代理类，最终会通过这个代理类去调用真正的类对象的成员函数
struct Show : pro::dispatch<void()>
{
  template <class T>
  void operator()(T& self) { self.show(); }
};

struct Model : pro::facade<Show> {};
 
class Who {
public: 
  void show() {
    std::cout << "model run!\n";
  }
};
 
class Boy {
public:
  void show() {
    std::cout << "boy run!\n";
  }
};
 
class Girl {
public:
  void show() {
    std::cout << "girl run!\n";
  }
};
 
class Man {
public:
  void show() {
    std::cout << "man run!\n";
  }
};
 
void justrun(pro::proxy<Model> m) { 
  m.invoke<Show>();
}
 
int main() {
  Who who;
  Girl girl;
  Boy boy;
  Man man;
 
  justrun(&who);
  justrun(&girl);
  justrun(&boy);
  justrun(&man);
}
```

此外，还可以用宏简化定义方式：
```C++
#include<iostream>
#include "proxy.h"

DEFINE_MEMBER_DISPATCH(Show, show, void());
DEFINE_FACADE(Model, Show);
```