[SWIG](https://www.swig.org/)（Simplified Wrapper and Interface Generator） 是一个开源的开发工具，用于将 C/C++ 代码与其他高级编程语言（如 Python、Java、C#、Ruby、Perl 等）进行连接。它的核心功能是自动生成“包装代码”（Wrapper Code），使得其他语言能够直接调用 C/C++ 的函数、类或数据结构，而无需手动编写复杂的绑定代码。

通过 SWIG，只需定义接口文件并生成包装代码，即可实现 C++ 到 C# 的无缝调用。此方法适用于快速集成现有 C/C++ 库到 C# 项目，但需注意平台兼容性和**内存管理**。


**核心功能**

1. **自动生成接口代码**：SWIG 通过解析 C/C++ 的头文件（`.h`），自动为目标语言生成对应的接口代码。例如：
    - 将 C++ 类包装为 Python 的类。
    - 将 C 函数暴露给 Java 或 C# 调用。
2. **支持多种语言**：SWIG 支持超过 20 种编程语言，包括：
    - **脚本语言**：Python、Ruby、Lua、Perl、PHP。
    - **静态类型语言**：Java、C#、Go、D。
    - **其他**：R、Tcl、OCaml 等。
3. **简化跨语言集成** 
    开发者无需深入理解目标语言的底层调用机制（如 Python 的 C API 或 Java 的 JNI），SWIG 会处理数据类型转换、内存管理和异常处理等细节。


**应用场景**

1. **为现有 C/C++ 库提供高级语言接口**  
    例如：将高性能的数学库（C/C++ 编写）封装成 Python 模块，供数据科学家使用。
2. **在脚本语言中嵌入 C/C++ 代码**  
    例如：用 Lua 编写游戏逻辑，通过 SWIG 调用 C++ 的图形渲染引擎。
3. **跨语言系统集成**  
    例如：将 C++ 的后端服务与 Java 的前端应用连接。


### 工作原理

1. **编写接口文件**（`.i` 或 `.swg`）：定义需要暴露给目标语言的 C/C++ 函数、类或变量。
```swig
// example.i
%module example
%{
#include "example.h"
%}
int add(int a, int b);
```

2. **运行 SWIG 生成包装代码** ：SWIG 解析接口文件，生成目标语言的包装代码（如 `example_wrap.cxx` 和 `example.py`）。

3. **编译并链接** ：将生成的包装代码与原始 C/C++ 代码一起编译，生成目标语言的扩展模块（如 Python 的 `.so` 或 `.pyd` 文件）。


### 示例

1. **C++ 代码** ：假设有一个简单的 C++ 函数和类需要暴露给 C#
```c++
// example.h
#pragma once

class Calculator {
public:
    int add(int a, int b);
    static double multiply(double x, double y);
};

// example.cpp
#include "example.h"

int Calculator::add(int a, int b) {
    return a + b;
}

double Calculator::multiply(double x, double y) {
    return x * y;
}
```

2. **SWIG 接口文件**（.i）：创建接口文件 `example.i`，定义需要暴露的内容：
```swig
// example.i
%module example

%{
#include "example.h"
%}

// 暴露 Calculator 类和它的方法
%include "example.h"
```


3. **生成包装代码** ：运行 SWIG 生成 C# 和 C++ 的包装代码（假设使用 Windows，安装 SWIG 并配置环境变量）
```bash
swig -c++ -csharp example.i
```

生成的文件：
- `example_wrap.cxx`：C++ 包装代码（用于编译动态库）。
- `example.cs`：C# 代理类（供 C# 调用）。
- `examplePINVOKE.cs`：C# 与 C++ 的底层交互代码（P/Invoke 封装）。


4. **编译 C++ 动态库** ：使用 Visual Studio 或命令行编译 C++ 代码为动态库（DLL）。

**方法 1：命令行编译（需安装 Visual Studio 工具链）**

```bash
cl /LD /EHsc /I. example.cpp example_wrap.cxx /link /OUT:example.dll
```

**方法 2：Visual Studio 项目**

- 创建一个 **C++ 动态链接库（DLL）** 项目。
- 添加 `example.h`、`example.cpp` 和 `example_wrap.cxx` 到项目中。
- 编译生成 `example.dll`。



5. **C# 调用代码** ：在 C# 项目中引用生成的 `example.cs` 和 `examplePINVOKE.cs`，并调用 C++ 代码：
```c#
// Program.cs
using System;

class Program {
    static void Main() {
        // 调用静态方法
        double result1 = Calculator.multiply(3.0, 4.0);
        Console.WriteLine($"3 * 4 = {result1}"); // 输出 12

        // 实例化对象并调用成员方法
        Calculator calc = new Calculator();
        int result2 = calc.add(5, 7);
        Console.WriteLine($"5 + 7 = {result2}"); // 输出 12
    }
}
```


6. **编译并运行 C# 项目**
	- **确保以下文件在同一目录**：
	    - `example.dll`（C++ 动态库）。
	    - `example.cs` 和 `examplePINVOKE.cs`（SWIG 生成的 C# 代码）。
	    - `Program.cs`（主程序）。
	- **编译 C# 代码**：
	```bash
	csc Program.cs example.cs examplePINVOKE.cs
	```
	- **运行程序**：
	```
	program.exe
	```


**扩展：处理 C++ 对象生命周期**

若 C++ 类需要手动管理内存，可在 SWIG 接口文件中添加析构函数的映射：

```swig
// example.i
%module example

%{
#include "example.h"
%}

// 添加析构函数的 C# 封装
%typemap(csdisposing) Calculator %{
    ~Calculator() {
        Dispose();
    }
%}

%include "example.h"
```