对于编译型语言来说，构建（Build）几乎是日常工作的一部分。

> 刚开始接触C++编译时，觉得可以直接使用 Visual Studio 编写和运行所有的 C++ 程序，后来渐渐了解到不同 CPU 的底层架构也不同，不同系统的编译器也不一致，渐渐了解了有多种编译器。
> 再后来，觉得使用编译器的编译命令可以完成一切编译，但是项目越来越大，编译命令也越来越复杂，各种库依赖需要添加到编译命令中，命令越来越长，感觉到不方便，然后才意识到构建工具的重要性。

## [GNU Make](构建/GNU%20Make.md)

GNU make 是一种流行的、常用的用于构建 C 语言软件的程序。用于构建 Linux 内核和其他常用的GNU/Linux 程序和软件库。

## [CMake](构建/CMake.md)

CMake 是一个工程文件生成工具，允许开发者编写一种平台无关的 `CMakeList.txt` 文件来定制整个编译流程，然后再根据目标用户的平台或者 IDE 进一步生成所需的本地化工程文件，如 Unix 的 Makefile 或 Windows 的 Visual Studio 解决方案。

## [Bazel](构建/Bazel.md)

[Bazel](https://bazel.google.cn/?hl=en) 是 Google 开源的编译构建工具，是一款类似于 Make、Maven 和 Gradle 的开源构建和测试工具，采用 Client/Server 运行模式，为云编译而生。

它使用可读的高级构建语言，支持多种编程语言编写的项目，并且能够为多个平台进行构建。Bazel 支持构建包含多个仓库、大量开发人员的大型代码库。

## [MSBuild](../../DevOps/Build/MSBuild.md)

MSBuild（Microsoft Build Engine）是用于构建 .NET 和 C++ 项目的平台和工具。它用于自动化构建过程，包括编译源代码、创建可执行文件或库、生成资源文件、运行单元测试、创建安装包等。

MSBuild 是 Visual Studio 内部的构建引擎，但也可以独立运行，不依赖于 Visual Studio。

## Meson

