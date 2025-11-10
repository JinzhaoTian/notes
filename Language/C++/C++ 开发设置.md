
# VS Code 开发环境设置

### 下载编译器

在Windows下，一般比较好的编译器就是 [MinGW-w64](https://www.mingw-w64.org/) ，但是**不推荐在官网下载**，可以去 [Github](https://github.com/niXman/mingw-builds-binaries/releases) 下载。

![](_imgs/Pasted%20image%2020240116171452.png)
下载后解压到一个目录下，然后**在系统环境中添加这个Path**。

### C/C++ 插件

1. 安装 C/C++ IntelliSense

![](_imgs/Pasted%20image%2020240116221102.png)



### 配置 C/C++ 环境

![](_imgs/Pasted%20image%2020240116224545.png)

需要配置：
1. 编译器路径
2. 包含路径
3. C标准：C17、C23
4. C++标准：C++17、C++20、C++23





### 重要文件

在使用VS Code进行C/C++的开发过程中，有三个至关重要的配置文件，分别是 **tasks.json**，**launch.json** 和 **c_cpp_properties.json**。

1. `tasks.json` 是在 vscode 中辅助程序编译的模块，可以代你执行类似于在命令行输入 `gcc hello.c -o hello` 命令的操作，你只要在图形界面下操作即可生成可执行文件。

2. `launch.json` 是用于运行 ( run ) 和调试 ( debug ) 的配置文件，可以指定语言环境，指定调试类型等等内容。

3. [c_cpp_properties.json](https://code.visualstudio.com/docs/cpp/c-cpp-properties-schema-reference) 主要用来设置包含头文件的路径，设置 C/C++ 支持的版本号等等。


常见变量名：

- `${workspaceFolder}` - VS Code当前打开工作区文件夹的路径
- `${file}` - 当前打开文件的绝对路径
- `${fileBasename}` - 当前打开文件的名称
- `${fileBasenameNoExtension}` - 当前打开文件的名称，但是不加后缀名
- `${fileDirname}` - 文件所在的文件夹路径




[使用VSCode和CMake构建跨平台的C/C++开发环境 - iwiniwin - 博客园 (cnblogs.com)](https://www.cnblogs.com/iwiniwin/p/13705456.html)