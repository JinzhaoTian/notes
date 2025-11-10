Clang 是 [LLVM](../../Compile%20Principle/LLVM.md) 项目的一个子项目，基于 LLVM 架构的 C / C++ / Objective-C 编译器**前端**。

![](imgs/Pasted%20image%2020240711235544.png)


相比于 GCC，Clang 具有如下优点：
- **编译速度快**：在某些平台上，Clang 的编译速度显著的快过 GCC（Debug 模式下编译 OC 速度比 GCC 快 3 倍）
- **占用内存小**：Clang 生成的 AST 所占用的内存是 GCC 的五分之一左右
- **模块化设计**：Clang 采用基于库的模块化设计，易于 IDE 集成及其他用途的重用
- **诊断信息可读性强**：在编译过程中，Clang 创建并保留了大量详细的元数据（metadata），有利于调试和错误报告
- 设计清晰简单，容易理解，易于扩展增强。


## 安装

### Windows

在 Windows 下 Clang 指令只是负责编译的前端工作，即识别编译指令，与提供报错提示，并不负责具体的代码编译工作，所以在 Windows 下光安装 LLVM 是不够的，还需要有实际的编译链接库，例如安装 MinGW 或者 MSVC 来获取需要的运行库。

- 下载页：[Release LLVM 20.1.8 · llvm/llvm-project](https://github.com/llvm/llvm-project/releases/tag/llvmorg-20.1.8)

