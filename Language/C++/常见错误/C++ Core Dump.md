在 C++ 中，core dump（核心转储）是指当程序异常终止时（例如由于段错误、总线错误、浮点异常等），操作系统将程序当时的内存状态、寄存器状态、堆栈指针等关键信息保存到一个文件中，这个文件通常称为 core 文件。**核心转储文件可以用于后续的调试，以确定程序崩溃的原因**。

## 组成

core dump 是程序崩溃时生成的一个文件，包含了程序在崩溃瞬间的完整内存映像，包括：
- **堆栈信息**
- **寄存器状态**
- **堆内存数据**
- **全局变量**
- **程序计数器等**

## 导致 Core Dump 的原因

1. **内存访问错误**
```cpp
// 空指针解引用
int* ptr = nullptr;
*ptr = 10;  // Core dump!

// 野指针
int* wild_ptr;
*wild_ptr = 5;  // Core dump!
```

2. **数组越界**
```cpp
int arr[5] = {1, 2, 3, 4, 5};
arr[10] = 100;  // 越界访问，可能导致 core dump
```

3. **栈溢出**
```cpp
void recursiveFunction() {
    int largeArray[100000];  // 栈空间不足
    recursiveFunction();     // 无限递归
}
```

4. **双重释放**
```cpp
int* ptr = new int(10);
delete ptr;
delete ptr;  // 重复释放，core dump!
```

5. **使用已释放的内存**
```cpp
int* ptr = new int(10);
delete ptr;
*ptr = 20;  // 使用已释放内存，core dump!
```

## 配置启用

在 Linux 中，
```bash
# 检查当前 core dump 设置
ulimit -c

# 启用 core dump（临时）
ulimit -c unlimited

# 永久启用，在 ~/.bashrc 或 /etc/security/limits.conf 中添加
ulimit -c unlimited

# 设置 core dump 文件路径和格式
echo "/tmp/core-%e-%p-%t" > /proc/sys/kernel/core_pattern
```

## 使用 Core Dump 进行调试

当你有 core 文件时，可以使用调试器（如 gdb ）来分析它。

假设你的程序名为 `my_program`，core 文件为 `core.1234`，你可以这样启动 gdb：
```bash
gdb my_program core.1234
```


1. **使用 GDB 分析**
```bash
# 编译时加入调试信息
g++ -g -o myprogram myprogram.cpp

# 使用 GDB 分析 core dump
gdb myprogram core
```

2. **GDB 常用命令**
```gdb
# 查看崩溃位置
bt
backtrace

# 查看变量值
print variable_name

# 查看寄存器
info registers

# 查看源代码位置
list
```

## 预防 Core Dump 的最佳实践

1. **智能指针管理内存**
```cpp
#include <memory>

std::unique_ptr<int> ptr = std::make_unique<int>(10);
// 自动管理内存，避免内存泄漏和重复释放
```

2. **边界检查**
```cpp
std::vector<int> vec = {1, 2, 3, 4, 5};
if (index < vec.size()) {
    vec[index] = value;
}
```

3. **异常处理**
```cpp
try {
    // 可能抛出异常的代码
    riskyOperation();
} catch (const std::exception& e) {
    std::cerr << "Error: " << e.what() << std::endl;
}
```

4. **使用静态分析工具**
```bash
# Valgrind 内存检查
valgrind --tool=memcheck ./myprogram

# AddressSanitizer
g++ -fsanitize=address -g -o myprogram myprogram.cpp
```

