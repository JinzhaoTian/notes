LLVM 源自 Chris Lattner 在 2000 年对 UUIC 的研究，其希望为所有静态和动态语言创建一种动态编译技术。LLVM 最初是 Low Level Virtual Machine 的缩写，现在 LLVM 已成为正式的商标名称，适用于 LLVM 下的所有项目，包括
- LLVM 中间表示（LLVM IR）
- LLVM 调试工具（LLDB Debugger）
- LLVM C/C ++ 标准库（libc，libc++）

LLVM 可用作传统的编译器，JIT 编译器，汇编器，调试器，静态分析工具，以及与编程语言相关的其他功能。LLVM 基础设施适用于若干 Unix 系统（GNU/Linux，FreeBSD ，Mac OS）和 Windows 系统。

LLVM 使用 BSD 开源许可。

## 历史

- 2003 年，发布 LLVM 1.0 版本 。
- 2005 年，Apple Inc. 聘请 Chris Lattner 及其团队为 Apple 计算机开发编程语言和编译器，此后 LLVM 的开发进入了快车道。从 LLVM 2.5 开始，每年都会发布两个次要的 LLVM 版本（通常在三月和九月）。
- 2011 年 11 月，LLVM 3.0 发布，成为默认的 XCode 编译器。默认情况下，XCode 5 开始使用Clang 和 LLVM 5.0 。版本策略针对 LLVM 5.0 和更高版本进行了调整，并且每年发布两个主要版本。



## 架构

传统的编译器架构分为前端（Frontend），优化器（Optimizer），后端（Backend）架构：

![](_imgs/Pasted%20image%2020250717174555.png)

- 前端（Frontend）：词法分析、语法分析、语义分析、生成中间代码。
- 优化器（Optimizer）：中间代码优化。
- 后端（Backend）：生成机器码。


LLVM 将架构进行扩展，

![](_imgs/Pasted%20image%2020250717174729.png)

不同的前端后端使用统一的中间代码 LLVM IR（ LLVM Intermediate Representation），将优化阶段变成了一个通用的阶段，它针对的是统一的 LLVM IR，不论是支持新的编程语言，还是支持新的硬件设备，都不需要对优化阶段做修改。如果需要支持一种新的编程语言，那么只需要实现一个新的前端。如果需要支持一种新的硬件设备，那么只需要实现一个新的后端。

相比之下，GCC 的前端和后端没分得太开，前端后端耦合在了一起。所以 GCC 为了支持一门新的语言，或者为了支持一个新的目标平台，就变得特别困难。

LLVM 现在被作为实现各种静态和运行时编译语言的通用基础结构（GCC 家族、Java、.NET、Python、Ruby、Scheme、Haskell、D等）。

## 支持语言

1. [Clang](Clang.md) ：C / C++ / Objective-C
2. LLDB： a next generation, high-performance debugger.


