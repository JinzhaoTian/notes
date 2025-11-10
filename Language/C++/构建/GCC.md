
GCC（GNU Compiler Collection）是 GNU 工具链的主要组成部分，是一套以 GPL 和 LGPL 许可证发布的程序语言编译器自由软件，由 Richard Stallman 于 1985 年开始开发。

GCC 原名为 GNU C 语言编译器，因为它原本只能处理 C 语言，但如今的 GCC 不仅可以编译 C、C++ 和 Objective-C，还可以通过不同的前端模块支持各种语言，包括 Java、Fortran、Ada、Pascal、Go 和 D 语言等等。


**语法**：

```
 gcc [options] file...
```

**选项**：

- `-pass-exit-codes` ：从一个阶段以最高错误代码退出。
- `--target-help` ：显示特定于目标的命令行选项。
- `--help={common|optimizers|params|target|warnings|[^]{joined|separate|undocumented}}[,...]` ：显示特定类型的命令行选项（使用 `-v --help` 显示子进程的命令行选项）。
- `-dumpspecs` ：显示所有内置规范字符串。
- `-dumpversion` ：显示编译器的版本。
- `-dumpmachine` ：显示编译器的目标处理器。
- `-print-search-dirs` ：显示编译器搜索路径中的目录。
- `-print-libgcc-file-name` ：显示编译器配套库的名称。
- `-print-file-name=<lib>` ：显示库 `<lib>` 的完整路径。
- `-print-prog-name=<prog>` ：显示编译器组件 `<prog>` 的完整路径。
- `-print-multiarch` ：显示目标的规范化 GNU 三元组，用作库路径中的一个组件。
- `-print-multi-directory` ：显示 libgcc 版本的根目录。
- `-print-multi-lib` ：显示命令行选项和多个库搜索目录之间的映射。
- `-print-multi-os-directory` ：显示操作系统库的相对路径。
- `-print-sysroot` ：显示目标库目录。
- `-print-sysroot-headers-suffix` ：显示用于查找标题的 sysroot 后缀。
- `-Wa,<options>` ：将逗号分隔的 `<options>` 传递给汇编器（assembler）。
- `-Wp,<options>` ：将逗号分隔的 `<options>` 传递给预处理器（preprocessor）。
- `-Wl,<options>` ：将逗号分隔的 `<options>` 传递给链接器（linker）。
- `-Xassembler <arg>` ：将 `<arg>` 传递给汇编器（assembler）。
- `-Xpreprocessor <arg>` ：将 `<arg>` 传递给预处理器（preprocessor）。
- `-Xlinker <arg>` ：将 `<arg>` 传递给链接器（linker）。
- `-save-temps` ：不用删除中间文件。
- `-save-temps=<arg>` ：不用删除指定的中间文件。
- `-no-canonical-prefixes` ：在构建其他 gcc 组件的相对前缀时，不要规范化路径。
- `-pipe` ：使用管道而不是中间文件。
- `-time` ：为每个子流程的执行计时。
- `-specs=<file>` ：使用 `<file>` 的内容覆盖内置规范。
- `-std=<standard>` ：假设输入源为 `<standard>`。
- `--sysroot=<directory>` ：使用 `<directory>` 作为头文件和库的根目录。
- `-B <directory>` ：将 `<directory>` 添加到编译器的搜索路径。
- `-v` ：显示编译器调用的程序。
- `-###` ：与 `-v` 类似，但引用的选项和命令不执行。
- `-E` ：仅执行预处理（不要编译、汇编或链接）。
- `-S` ：只编译（不汇编或链接）。
- `-c` ：编译和汇编，但不链接。
- `-o <file>` ：指定输出文件。
- `-pie` ：创建一个动态链接、位置无关的可执行文件。
- `-I` ：指定头文件的包含路径。
- `-L` ：指定链接库的包含路径。
- `-shared` ：创建共享库/动态库。
- `-static` ：使用静态链接。
- `--help` ：显示帮助信息。
- `--version` ：显示编译器版本信息。

**常用选项**：
- `-g` ：在目标文件中嵌入调试信息，以便 gdb 调试
- `-f` ：



- `-L` ：指定静态链接库所在的路径。
- `-l` ：指定静态链接库的名称，此处需要省略名称中的 `lib` 和 `.a` 后缀。如，对于 OpenGL 的窗口静态库 `libglfw3.a` ，在编译时要加上 ：
```bash
gcc [...] -L ${workspaceFolder}/dependencies/lib -lglfw3
```

## 发展历程

理查德·斯托曼（Richard Stallman）于1984年发起了 GNU 工程，以构建一个类似 UNIX 的开源软件系统。随着时间的流逝，GNU 操作系统尚未广泛发展。但是，它已经孵化了许多出色且有用的开源软件工具，例如 Make，Sed，Emacs，Glibc，GDB 和 GCC 。这些 GNU 开源软件和 Linux 内核共同构成了 GNU / Linux 系统。最初，GCC 为 CNU 语言提供了基于 C 编程语言的稳定可靠的编译器。它的全名是 GNU C 编译器。后来，支持了更多的语言（例如Fortran，Obj-C和Ada），并且 GCC 的全名更改为GNU Compiler Collection。

GCC-1.0 由理查德·斯托曼（Richard Stallman）于1987年发布，距今已有30多年的历史。从软件角度来看，它已经很老了。有人收集了 1989 年至 2012 年之间的 GCC 开发记录，并制作了一个三十分钟的动画视频（1989-2012年GNU Compiler Collection开发历史），直观地展示了GCC的开发过程。我们可以从GCC的版本中了解其发展历史：

- GCC-1.0：由 Richard Stallman 在1987年发布。
- GCC-2.0：1992 年发布并支持 C++。后来，GCC 社区分裂了，因为 Richard Stallman 将 GCC 定义为 GNU 系统的可靠 C 编译器，并认为当时的 GCC 对于 GNU 系统已经足够了，开发重点应从 GCC 转移到 GNU 系统本身。其他主要开发商希望继续改善 GCC，并在各个方面做出更根本的发展。这些活跃的开发人员于 1997 年离开 GCC 社区，并开发了 EGCS fork。
- GCC-3.0：显然，开发人员通常对好的编译器有强烈的渴望。EGCS 分支发展顺利，并得到越来越多开发人员的认可。最终，EGCS 被用作新的 GCC 主干，并于 2001 年发布了 GCC-3.0 。分裂的社区再次被合并，但是 Richard Stallman 的影响力在一定程度上被削弱了。此外，海湾合作委员会工业委员会已开始决定海湾合作委员会的发展方向。
- GCC-4.0：2005 年发布，此版本已集成到树串行存储体系结构（SSA）中，并且 GCC 演变为现代编译器。
- GCC-5.0：2015 年发布，GCC 版本政策进行了调整，并且每年都会发布主要版本。意外的好处是版本号与年份相对应。例如，GCC-7 于 2017 年发布，GCC-9 于 2019 年发布。

现在，面对 LLVM 的竞争压力，GCC 社区积极进行了许多调整，例如加快编译速度和改进编译警告信息。在过去的 30 年中，GCC 已从编译器行业的挑战者发展成为 Linux 系统的主流编译器，现在正面临 LLVM 的挑战。








