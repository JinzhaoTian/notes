
## Code Style

[Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html)

[C++ 风格指南 - 内容目录 — Google 开源项目风格指南 (zh-google-styleguide.readthedocs.io)](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/contents/)

[styleguide/cpplint/cpplint.py at gh-pages · google/styleguide (github.com)](https://github.com/google/styleguide/blob/gh-pages/cpplint/cpplint.py)

[7. 命名约定 — Google 开源项目风格指南 (zh-google-styleguide.readthedocs.io)](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/naming/)


## 命名规范

### 匈牙利命名法

匈牙利命名法是早期的规范，由微软的一个匈牙利人发明的，是 IDE 还十分智障的年代的产物。那个年代，当代码量很多的时候，想要确定一个变量的类型是很麻烦的，不像现在 IDE 都会给提示，所以才产生了这样一个命名规范，一个十分系统却又琐碎的命名规范。

该命名规范，要求前缀字母用变量类型的缩写，其余部分用变量的英文或英文的缩写，单词第一个字母大写。

```C++
int iMyAge;        #  "i": int
char cMyName[10];  #  "c": char
float fManHeight;  #  "f": float
```

前缀类型还有：
```C++
a      数组（Array）
b      布尔值（Boolean）
by     字节（Byte）
c      有符号字符（Char）
cb     无符号字符（Char Byte，并没有神马人用的）
cr     颜色参考值（Color Ref）
cx,cy  坐标差（长度 Short Int）
dw     双字（Double Word）
fn     函数（Function）
h      Handle（句柄）
i      整形（Int）
l      长整型（Long Int）
lp     长指针（Long Pointer）
m_     类成员（Class Member）
n      短整型（Short Int）
np     近程指针（Near Pointer）
p      指针（Pointer）
s      字符串（String）
sz     以 Null 做结尾的字符串型（String with Zero End）
w      字（Word）
```

### 驼峰式命名法

驼峰式命名法，又叫小驼峰式命名法，要求第一个单词首字母小写，后面其他单词首字母大写，简单粗暴易学易用。

```C++
int myAge;
char myName[10];
float manHeight;
```


### 帕斯卡命名法

帕斯卡命名法与小驼峰式命名法的最大区别在于，每个单词的第一个字母都要大写。

```C++
int MyAge;
char MyName[10];
float ManHeight;
```


### 下划线命名法

下划线命名法，要求单词与单词之间通过下划线连接即可。

```C++
int my_age;
char my_name[10];
float man_height;
```



## 头文件最佳实践顺序

推荐的头文件包含顺序如下，**从上到下，优先级递减**：
1. **关联的头文件**（The associated header file）
2. **C 系统头文件**（C system headers）
3. **C++ 系统头文件**（C++ system headers）
4. **其他库的头文件**（Other libraries' headers）
5. **本项目内的头文件**（Your project's headers）

```cpp
// 1. 关联的头文件 (自身对应的头文件)
// 这是第一条规则，用于验证该头文件是否自包含（不依赖其他include顺序也能编译）。
#include "my_class.h"

// 2. C 系统头文件 (使用角括号，无扩展名或.h)
#include <sys/types.h>
#include <unistd.h>

// 3. C++ 系统头文件 (使用角括号，通常无.h后缀)
#include <vector>
#include <string>
#include <iostream>

// 4. 其他第三方库的头文件 (使用角括号或引号，取决于安装方式)
#include <gtest/gtest.h> // Google Test
#include <boost/algorithm/string.hpp> // Boost

// 5. 本项目内的其他头文件 (使用引号)
#include "utils/helper_functions.h"
#include "config/project_settings.h"
```

> [!tip] 为什么这个顺序是最佳的？
> 1. **自包含性检查（Self-containment Check）**：
> 	- 将关联的头文件放在第一位是一个防守性编程技巧，如果 `my_class.h` 本身不完整，它需要包含其他头文件才能编译成功（这是一种不好的实践）。如果它被放在第一位，你在编译 `my_class.cpp` 时会立即发现这个错误。如果把它放在后面，可能会因为前面的头文件恰好定义了所需的内容而掩盖了这个错误，导致项目其他部分包含该头文件时编译失败。
> 2. **从最严格到最宽松（Most Specific to Least Specific）**：
> 	- 系统头文件和第三方库头文件通常是稳定、不可修改的，是你的代码所依赖的基础。
> 	- 将本项目头文件放在最后，是因为它们最可能被频繁修改，并且其内容最可能与你当前编写的源文件相关。这个顺序减少了本项目头文件对系统库的意外影响。
> 3. **避免隐藏的依赖（Hidden Dependencies）**：
> 	- 这个顺序最大限度地减少了因头文件包含顺序而导致的隐藏依赖。如果一个头文件 `a.h` 需要另一个头文件 `b.h` 才能正常工作，那么 `a.h` 应该自己包含 `b.h`，而不是依赖包含它的 `.cpp` 文件以正确的顺序去包含。将关联头文件放在第一位强制实施了这一规则。
> 4. **减少编译时间（可能）**：
> 	- 现代编译器通常使用预编译头文件（Precompiled Headers, PCH）来极大地加速编译，而 PCH 的成功运用严重依赖于稳定的、一致的包含顺序。遵循这个规则使得使用 PCH 更容易。

### 其他重要规则

1. **仅包含必需的头文件**：不要包含用不到的头文件，这会增加不必要的编译依赖和编译时间。
2. **使用 Include Guards 或 `#pragma once`**：每个头文件都必须有防止重复包含的机制。
	- `#pragma once`（大多数现代编译器都支持，简洁高效）
	- Include Guards（标准方式，所有编译器都支持）
```cpp
#ifndef MY_UNIQUE_HEADER_NAME_H_
#define MY_UNIQUE_HEADER_NAME_H_
// ... 头文件内容 ...
#endif // MY_UNIQUE_HEADER_NAME_H_
```
3. **在头文件中尽量使用前向声明（Forward Declaration）**：
	- 如果类 `ClassA` 仅被用作指针或引用（不需要知道其大小或成员），那么在头文件中使用 `class ClassA;` 进行前向声明，而不是包含 `class_a.h`。
	- 这可以**切断编译依赖，减少编译时间**。
```cpp
// my_class.h
#pragma once
class OtherClass; // 前向声明， good!

class MyClass {
public:
    void doSomething(OtherClass* ptr); // 只需要指针，不需要包含OtherClass的定义
// ...
};
```