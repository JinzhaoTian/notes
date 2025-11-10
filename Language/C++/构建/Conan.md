
## 核心功能

1. **依赖管理**
    - 自动下载、编译和链接项目所需的依赖库。
    - 支持版本控制，可以指定依赖的版本范围（如 `zlib/1.2.11`）。

2. **跨平台支持**
    - 支持 Windows、Linux、macOS 等操作系统。
    - 处理不同平台、编译器（如 GCC、Clang、MSVC）、架构（x86、ARM）的兼容性问题。

3. **二进制包管理**
    - 预编译二进制包（避免重复编译）。
    - 支持自定义编译选项（如静态库/动态库、Debug/Release 模式）。

4. **与构建系统集成**
    - 兼容 CMake、Makefile、Visual Studio、Meson 等构建工具。
    - 生成对应的构建配置文件（如 `conanbuildinfo.cmake`）。

5. **私有仓库支持**
    - 可以搭建企业内部的私有包仓库（Artifactory 或 Conan 官方远程仓库）。


## 基本概念

- **Package（包）**：一个库（如 OpenSSL、Boost）及其元数据（版本、依赖、编译选项等）。
- **Recipe（配方）**：定义如何构建包的脚本（`conanfile.py` 或 `conanfile.txt`）。
- **Remote（远程仓库）**：包的存储服务器（默认是 Conan Center，类似 pip 的 PyPI）。
- **Profile（配置）**：指定平台、编译器、编译选项等（如 `gcc 9.3` + `C++17`）。


## 适用场景

- 企业级私有依赖管理（如NASA JPL、Siemens）
- 跨平台项目（Windows/Linux/macOS/嵌入式）


