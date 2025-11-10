CMake 是一个跨平台的自动化构建工具，用于管理软件编译过程。它允许开发者编写简单的配置文件 `CMakeList.txt` 来定制整个编译流程，然后再根据目标用户的平台或者 IDE 进一步生成所需的本地化工程文件，如 Unix 的 Makefile 或 Windows 的 Visual Studio 解决方案。

> [!info]
> ***Write once, run everywhere*** .

![](imgs/Pasted%20image%2020250716170846.png)

## 核心功能

1. **跨平台支持**
    - 支持 Windows、Linux、macOS 等操作系统。
    - 可生成多种构建系统的文件（如 Makefile、Ninja、Xcode 项目、Visual Studio 解决方案等）。

2. **简化构建流程**：开发者只需编写 `CMakeLists.txt` 文件描述项目结构，无需手动编写复杂的构建脚本。

3. **依赖管理**
    - 自动检测系统环境（如编译器、库路径）。
    - 支持查找第三方库（如 OpenCV、Boost）并集成到项目中。

4. **模块化与复用**
    - 支持模块化配置，可管理大型项目的子模块。
    - 允许通过 `find_package()` 复用外部库。


## 核心概念

### 变量

1. **变量类型**
	- **普通变量**：`set(VAR_NAME value)`
	- **缓存变量**：跨多次构建保留（通过 `-D` 传递）：`set(USE_FEATURE_X ON CACHE BOOL "Enable feature X")`
	- **环境变量**：`$ENV{PATH}`，捕获系统中设置的环境变量或者临时环境变量

2. **作用域**
	- **函数作用域**：`function()` 内定义的变量局部有效。
	- **目录作用域**：`CMakeLists.txt` 中定义的变量对子目录可见（需用 `PARENT_SCOPE` 向上传递）。

#### CMake 变量

1. **`CMAKE_TOOLCHAIN_FILE`**：CMake 中用于交叉编译（Cross-Compilation）的最核心、最重要的变量，其值是一个指向工具链文件（Toolchain File）的路径。
	- **工具链文件**：是一个纯文本的 `.cmake` 文件，其核心目的是在真正开始配置和构建项目之前，提前设置所有与编译工具链相关的变量，而不是使用 CMake 默认的本地的编译工具链。

> [!tip]
> `CMAKE_TOOLCHAIN_FILE` 虽然最初是为交叉编译设计的，但其机制非常通用，现在也被用于其他目的，最著名的例子是 `vcpkg`。
> 
> 在使用 `vcpkg` 时，它会提供一个工具链文件（`vcpkg.cmake`），在配置时指定这个文件：
> ```cmake
> cmake -B build -DCMAKE_TOOLCHAIN_FILE=/path/to/vcpkg/scripts/buildsystems/vcpkg.cmake
> ```
> 
> 这个工具链文件会引导 CMake 的 `find_package()` 等命令去 `vcpkg` 的安装目录中查找库，而不是系统的默认路径。
> 
> 这完美地展示了工具链文件的核心思想：提前改变 CMake 的默认查找和行为路径。



2. **`CMAKE_EXPORT_COMPILE_COMMANDS`**：设置是否生成**编译数据库** `compile_commands.json` 文件
	- `compile_commands.json`：是一个包含了项目中每个源文件编译时所用到的完整命令行指令的清单，对于每个源文件（如 `.cpp` 或 `.c` 文件），它都会记录一条信息，包括：
		- 编译该文件所在的目录（directory）
		- 要编译的源文件路径（file）
		- 用于编译该文件的完整命令（command）
	- 这个文件不是由 CMake 直接生成的，而是当 CMake 被配置为生成器（如 Ninja 或 Makefile）时，**由这些生成器在构建过程中创建出来的**。
		- 主要适用于 Makefile 和 Ninja 等 Generator，Visual Studio Generator **默认不支持**生成 `compile_commands.json` 文件。

3. **系统检测**：
	- **主机系统检测**：
		- `CMAKE_HOST_SYSTEM_NAME`：主机系统名称（Windows，Linux，Darwin）
		- `CMAKE_HOST_WIN32`：如果当前运行 CMake 的机器是 Windows 系统
		- `CMAKE_HOST_UNIX`：如果当前运行 CMake 的机器是类 Unix 系统
		- `CMAKE_HOST_LINUX`：如果当前运行 CMake 的机器是 Linux 系统
			- 3.25 版本以上可用
		- `CMAKE_HOST_BSD`：如果当前运行 CMake 的机器是 BSD 系统
		- `CMAKE_HOST_APPLE`：如果当前运行 CMake 的机器是 Apple 系统
	- **目标系统检测**：
		- `WIN32`：如果编译目标平台是 Windows 系统
		- `UNIX`：如果编译目标平台是类 Unix 系统
		- `LINUX`：如果编译目标平台是类 Linux 系统
			- 3.25 版本添加
		- `BSD`：如果编译目标平台是 BSD 系统
			- 3.25 版本添加
		- `APPLE`：如果编译目标平台是 Apple 系统
		- `ANDROID`：如果编译目标平台是 Android 系统
			- 3.7 版本添加
		- `IOS`：如果编译目标平台是 iOS 系统
			- 3.14 版本添加
		- `BORLAND`


### 依赖管理

1. **`find_package()`**：查找并加载一个外部库（包）的配置，从而在你的 CMake 项目中使用它。
```cmake
find_package(OpenCV REQUIRED)
```
成功运行后，它通常会提供：
- `<PackageName>_FOUND`：一个变量，指示是否找到了该包。
- 导入目标（Import Targets）（现代方式）（推荐）：
	- 如 `OpenCV::OpenCV`，你可以直接将其链接到你的目标，`target_link_libraries(your_target PRIVATE OpenCV::OpenCV)`。
	- CMake 会自动的将对应的头文件目录添加到 `your_target` 的编译环境中。
- 变量（传统方式）：
	- `<PackageName>_INCLUDE_DIRS`（头文件路径）和 `<PackageName>_LIBRARIES`（库文件路径），你需要手动将它们添加到 `include_directories()` 和 `target_link_libraries()` 中。

`find_package()` 有两种工作模式，它们决定了 CMake 如何查找包，CMake 会优先尝试 Module 模式，如果失败，则回退到 Config 模式。

>[!info]- Module 模式
> 这种模式依赖于一个名为 `Find<PackageName>.cmake` 的**查找模块（Find Module）** 文件，这个文件不是由库的开发者提供的，而是由 CMake 本身或你的项目提供的。
> 
> **工作原理**
> 1. CMake 会在 `CMAKE_MODULE_PATH` 所指定的路径中查找 `Find<PackageName>.cmake` 文件。
> 2. 首先，它会检查内置的模块目录（`<CMakeInstallation>/share/cmake-<version>/Modules`），这里包含了 CMake 官方提供的查找上百种常见库的模块（如 `FindOpenSSL.cmake`, `FindPNG.cmake`）。
> 3. 如果没找到，它会查看你通过 `list(APPEND CMAKE_MODULE_PATH /some/path)` 添加的路径。
> 4. 如果找到了 `Find<PackageName>.cmake` 文件，CMake 会执行它。
> 5. 这个 `*.cmake` 文件内部通常包含了查找头文件、库文件的逻辑（使用 `find_path()`，`find_library()` 等命令），并设置相应的 `*_FOUND`, `*_INCLUDE_DIRS`, `*_LIBRARIES` 等变量。

> [!info]- Config 模式
> 这种模式依赖于一个由库的开发者本身提供并随库一起安装的配置文件，是现代 CMake 的首选方式。
> 
> **工作原理**
> 1. CMake 会构造一个可能的路径列表来查找名为 `<PackageName>Config.cmake` 或 `<package-name>-config.cmake` 的文件。
> 2. 查找路径的优先级如下（简化版）：  
> 	- 由 `find_package()` 的 `PATHS` 选项显式指定。  
> 	- 一系列以 `CMAKE_PREFIX_PATH`、`CMAKE_FRAMEWORK_PATH`、`CMAKE_APPBUNDLE_PATH` 为前缀的路径。**`CMAKE_PREFIX_PATH` 是你最需要关心的**，你可以通过它来提示 CMake 查找第三方库的位置。  
> 	- 环境变量 `PATH` 中的目录（在 Windows 或 macOS 的 APP 包上会稍有变化）。
> 3. 如果找到了 `*Config.cmake` 文件，CMake 会执行它。因为这个文件是由库的作者编写的，所以它精确地知道该库的所有组件、目标和使用要求。

通过参数显式指定模式：
- `find_package(<PackageName> MODULE)`：只使用 Module 模式。
- `find_package(<PackageName> CONFIG)`：只使用 Config 模式。
- `find_package(<PackageName>)`：两种都尝试，先 Module 后 Config。


2. **`FetchContent`** / **`ExternalProject`**：动态下载和管理外部依赖（如 GitHub 项目）。
```cmake
include(FetchContent)
FetchContent_Declare(
  googletest
  URL https://github.com/google/googletest/archive/v1.13.0.zip
)
FetchContent_MakeAvailable(googletest)
```

### Target

在 CMake 中，一个 Target（目标）代表了一个构建系统要生成的最终产出物，或者是一个由自定义命令组成的集合。它不是一个文件，而是一个具有名称、属性和命令的抽象对象。

#### 常见类型

1. **可执行文件（Executable）**：
	- **创建命令**：`add_executable()`
    - **最终产出**：一个可以直接运行的程序（如 `.exe` 或无后缀的可执行文件）。

2. **库（Library）**：
    - **创建命令**：`add_library()`
    - **最终产出**：一个供其他目标使用的库文件（如 `.a`, `.lib`, `.so`, `.dll` 等）。
    - **类型**：
        - `STATIC`：静态库
        - `SHARED`：动态库/共享库
        - `INTERFACE`：接口库（一种特殊的库，本身不编译任何代码，只用来传递依赖关系）

3. **自定义目标（Custom Target）**
    - **创建命令**：`add_custom_target()`
    - **最终产出**：不产出任何文件，而是代表一组你希望执行的命令（例如：运行测试、打包发布包、生成文档、安装等），你可以通过 `make my_custom_target` 来显式执行这些命令。

#### 目标属性

所有属性都应该是 Target 的属性，并且依赖关系在 Target 之间显式地传递。

每个 Target 都自带一系列属性，你可以精确地控制它：
1. **包含路径**：
	- **命令**：`target_include_directories()`
2. **链接库**：
	- **命令**：`target_link_libraries()`
3. **编译选项**：
	- **命令**：`target_compile_options()`
4. **编译定义**：
	- **命令**：`target_compile_definitions()`

#### 依赖传递

Target 属性通过**访问控制**来传递依赖：
- **`PUBLIC`**：依赖者自动获得 Target 的功能属性（如头文件路径、编译定义、Target 所链接的库的）。
- **`PRIVATE`**：仅 Target 自身使用（如实现细节）。
- **`INTERFACE`**：仅依赖者使用（如头文件库）。

> [!tip] 依赖控制决策流程
> 你可以通过问一个问题来做决定：**“我的库的使用者需要知道这个第三方库的存在吗？”**
> 1. **你的目标类型是可执行文件（`add_executable`）？**
> 	- **99% 的情况用 `PRIVATE`**。因为没有人会链接你的可执行文件，没有传递依赖的必要。
> 2. **你的目标类型是库（`add_library`）？**
>    检查第三方库的使用位置：
> 	- **仅在 `.cpp` 源文件中使用 → `PRIVATE`**
> 		- _“我的使用者不需要知道它。”_
> 	- **在公共头文件（`.h`/`.hpp`）中也使用了 → `PUBLIC`**
> 		- _“我的使用者必须也能找到并链接这个库，否则代码编译不过。”_
> 	- **第三方库是纯头文件库，并且你的库是 `INTERFACE` 库 → `INTERFACE`**
> 		- _“我自己不编译，但我的使用者需要它。”_


#### 声明依赖

```cmake
add_dependencies(<target> [<target-dependency>]...)
```

是 CMake 中用于**显式声明目标间依赖关系**的命令，它确保在构建某个目标之前，其所依赖的其他目标已经被构建。

1. **自定义目标间的依赖**
```cmake
# 创建自定义目标
add_custom_target(generate_files
    COMMAND echo "Generating files..."
)

add_custom_target(process_files
    COMMAND echo "Processing files..."
)

# 声明依赖关系：process_files 依赖于 generate_files
add_dependencies(process_files generate_files)
```

2. **确保代码生成在编译前完成**
```cmake
# 代码生成目标
add_custom_target(generate_code
    COMMAND python generate_sources.py
    BYPRODUCTS generated.cpp
)

# 可执行文件
add_executable(my_app main.cpp generated.cpp)

# 确保生成代码在编译前完成
add_dependencies(my_app generate_code)
```

3. **处理非链接依赖**
```cmake
add_library(my_lib src1.cpp src2.cpp)
add_executable(my_app main.cpp)

# 即使没有链接关系，也声明依赖
add_dependencies(my_app my_lib)
```

> [!tip]- 应用场景
> **场景1**：**多步骤构建过程**
> ```cmake
> add_custom_target(step1
>     COMMAND echo "Step 1: Data preparation"
> )
> 
> add_custom_target(step2
>     COMMAND echo "Step 2: Data processing"
> )
> add_custom_target(step3
>     COMMAND echo "Step 3: Final compilation"
> )
> 
> # 定义执行顺序
> add_dependencies(step2 step1)
> add_dependencies(step3 step2)
> ```
> 
> **场景2**：**确保资源文件就绪**
> ```cmake
> # 下载资源文件
> add_custom_target(download_assets
>     COMMAND wget -O assets.zip http://example.com/assets.zip
>     COMMAND unzip assets.zip
> )
> 
> add_executable(game main.cpp)
> 
> # 确保资源下载完成后再编译
> add_dependencies(game download_assets)
> ```

> [!tip] 最佳实践
> 在现代 CMake 中，优先使用 `target_link_libraries` 来声明依赖，因为它会自动处理构建顺序；`add_dependencies` 主要用于声明那些不涉及链接但需要特定构建顺序的目标间依赖关系。

#### 链接控制

`target_link_libraries()` 命令**本身不直接区分动态或静态链接**，而是管理目标（可执行文件或库）之间的依赖关系，真正的链接类型主要由库文件本身的性质（是 `.a` 还是 `.so`，是 `.lib` 还是 `.dll`）以及编译器的设置决定。

| 特性                             | MSVC (Windows)                | GCC/Clang (Linux/macOS)                       |
| ------------------------------ | ----------------------------- | --------------------------------------------- |
| **静态库后缀**                      | `.lib`                        | `.a`                                          |
| **动态库导入库后缀**                   | `.lib` (与静态库相同)               | 通常不需要专门的导入库                                   |
| **动态库实现后缀**                    | `.dll`                        | `.so` (Linux), `.dylib` (macOS)               |
| **默认查找顺序**                     | 通常优先查找 `.lib`（无法仅通过名字区分静态/动态） | 通常优先查找 `.so`（动态库），除非指定 `-static`              |
| **显式指定静态库**                    | 直接指定完整的静态库文件名（如 `mylib.lib`）  | 直接指定完整的静态库文件名（如 `libmylib.a`）或使用 `-static` 选项 |
| **`target_link_libraries` 行为** | 处理传递的依赖关系，确保正确的链接库顺序和编译器选项    | 类似，处理依赖关系，传递必要的链接器选项和路径                       |

在 CMake 中，有多种方式来影响链接器最终选择动态库还是静态库：
1. **直接指定库文件全名**：如果直接指定了库文件的全路径（包括文件名），CMake 会直接使用你指定的这个文件：
```cmake
target_link_libraries(MyApp
    /path/to/libfoo.a  # 明确链接静态库（GCC/Clang）
    /path/to/bar.lib   # 明确链接静态库（MSVC）
)
```

2. **使用 `find_library()`**：可以通过 `find_library()` 指定完整的库名或 `NAMES` 来查找特定类型的库：
```cmake
find_library(FOO_LIBRARY NAMES libfoo.a foo) # 优先查找静态库
find_library(BAR_LIBRARY NAMES bar) # 查找系统默认优先的库（很可能是动态库）
if(FOO_LIBRARY)
    target_link_libraries(MyApp PRIVATE ${FOO_LIBRARY})
endif()
```

3. **链接器选项与标志**：虽然不推荐直接粗暴地设置 `-static`（因为它会影响所有链接的库），但你有时可能需要传递特定的链接器选项：
```cmake
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU")
    target_link_options(MyApp PRIVATE -Wl,-Bstatic) # 告诉链接器接下来尝试静态链接
    target_link_libraries(MyApp PRIVATE foo)        # 链接 libfoo
    target_link_options(MyApp PRIVATE -Wl,-Bdynamic) # 恢复动态链接，避免影响后续系统库
endif()
```
对于 MSVC，静态链接运行时库通常通过编译选项（如 `/MT` 或 `/MTd`）控制，而不是链接选项。这些选项可以使用 `target_compile_options` 设置。

4. **设置 `BUILD_SHARED_LIBS` 全局变量**：这个变量会影响项目中所有未显式指定类型的 `add_library` 命令。默认是 `OFF`，即默认生成静态库：
```cmake
set(BUILD_SHARED_LIBS ON) # 此后未指定类型的 add_library 默认生成动态库
```

5. **导入的目标（Imported Targets）**：对于第三方库，特别是通过 `find_package()` 找到的，它们通常提供导入的目标（如 `OpenSSL::SSL`）。这些目标已经预定义了它们所有的依赖关系，你只需要直接链接它们即可，CMake 会自动处理细节。
```cmake
find_package(OpenSSL REQUIRED)
target_link_libraries(MyApp PRIVATE OpenSSL::SSL)
```


### Generator

在 CMake 中，Generator（生成器）是指 CMake 用来生成特定构建系统文件的工具。它决定了 CMake 如何将 `CMakeLists.txt` 中的配置转换为目标平台上的构建脚本（如 Makefile、Visual Studio 项目文件等）。

1. **单配置生成器**（Single-Config Generators）
	- **Unix Makefiles**：生成标准的 `Makefile`
		- 适用于类 Unix 系统，如 Linux、macOS、FreeBSD
		- 通常与 [GCC](GCC.md) 或 [Clang](Clang.md) 编译器配合使用
	- **MinGW Makefiles**：生成适用于 `mingw32-make` 的 `Makefile` ，允许在 Windows 上使用类 Unix 的构建流程
		- 适用于 Windows 系统，搭配 MinGW 工具链
		- 通常与 [MinGW](MinGW.md) 编译器配合使用
	- **Ninja**：生成 `build.ninja` 文件，适用于追求高速构建的场景。

2. **多配置生成器**（Multi-Config Generators）
	- **Visual Studio**：生成 Visual Studio 的解决方案文件（`.sln`）
	- **Xcode**：生成 Xcode 项目文件（`.xcodeproj`）

## 命令行命令

#### 项目配置

```bash
cmake [options] <source-path>
```

**常用选项（options）**：
- `-G <generator>`：指定生成器（如 `"Unix Makefiles"`、`"Visual Studio 17 2022"`）
- `-D <var>=<value>`：设置 CMake 变量（缓存变量）
	- 如设置环境变量：`cmake -DVCPKG_ROOT=/path/to/vcpkg -B build -S .`
- `-B <build-path>`：指定构建目录（代替手动 `mkdir build && cd build`）。
- `-S <source-path>`：指定源码目录（CMakeLists.txt 所在路径）。


#### 项目构建

```bash
cmake --build <build-dir> [options]
```

**常用选项（options）**：
- `--config <Debug|Release>`：指定构建类型（多配置生成器如 Visual Studio 需使用）。
- `--target <target-name>`：仅构建指定目标（如 `my_app`）。
- `-j <N>` 或 `--parallel <N>`：并行编译（多线程加速）。


#### 项目安装

执行安装步骤（需在 `CMakeLists.txt` 中定义 `install()` 规则）
```bash
cmake --install <build-dir> [options]
```

**常用选项（options）**：
- `--prefix <install-path>`：指定安装目录（默认通常为 `/usr/local` 或 `C:\Program Files`）。
- `--config <Debug|Release>`：指定安装的构建配置。


#### 测试项目

运行项目中定义的测试（需先通过 `enable_testing()` 和 `add_test()` 配置），
```bash
ctest [options]
```

**常用选项（options）**：
- `-C <Debug|Release>`：指定测试的构建配置。
- `--output-on-failure`：测试失败时打印输出。
- `-R <regex>`：运行匹配正则表达式的测试用例。
- `-j <N>`：并行运行测试。


#### 清理构建

```bash
cmake --build <build-dir> --target clean
```

清理构建生成的中间文件（调用底层工具的 `clean` 目标）。




## `CMakeLists.txt`

`CMakeLists.txt` 的语法比较简单，由命令、注释和空格组成，其中命令是不区分大小写的。符号 `#` 后面的内容被认为是注释。命令由命令名称、小括号和参数组成，参数之间使用空格进行间隔。

假设工程文件如下：
```
Demo/
  ├─ main.cc
  ├─ MathFunctions.cc
  └─ MathFunctions.h
```


则简单的 CMakeLists.txt 可以为：
```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 设置C++版本
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

# 项目信息
project (Demo)

# 指定生成目标
add_executable(Demo main.cc MathFunctions.cc)
```


### 命令

1. **指定**运行此配置文件所需的 CMake 的**最低版本**
```cmake
cmake_minimum_required(VERSION <min>[...<policy_max>] [FATAL_ERROR])
```

2. **指定项目的名称**
```cmake
project(<PROJECT-NAME> [<language-name>...])
```

3. **设置变量**
```cmake
set(<variable> <value>... [PARENT_SCOPE])
```
设置变量的值，常见的有设置 C++ 版本，设置源文件。


4. **生成可执行文件**
```cmake
add_executable(<name> [WIN32] [MACOSX_BUNDLE] [EXCLUDE_FROM_ALL] [source1] [source2 ...])
```
将名为 `[source1] [source2 ...]` 的源文件编译成一个名称为 `<name>` 的可执行文件。

5. **生成静态库（`STATIC`）或动态库（`SHARED`）**
```cmake
add_library(<name> STATIC lib.cpp)
```

6. **设置全局 include 路径**
```cmake
include_directories()
```
所有在 `include_directories` 之后定义的目标（Target）都会继承这些包含路径，适用于简单项目的全局设置，但可能会意外影响多个目标。

7. **设置特定目标的 include 路径**
```cmake
target_include_directories(<target> [INTERFACE|PUBLIC|PRIVATE] [items...])
```
设置特定目标的包含路径，只对某个目标或一组目标生效。更适合复杂项目，提供细粒度的控制，可以指定包含路径的可见性，
- **PRIVATE**: 仅对该目标的源文件可见，使用它编译该目标时包含的目录。
- **PUBLIC**: 该目标和链接该目标的依赖目标都可见。
- **INTERFACE**: 仅对链接该目标的依赖目标可见，该目标本身不会使用这些包含目录。

7. **设置 link**
```cmake
target_link_libraries(<target> [INTERFACE|PUBLIC|PRIVATE] [items...])
```

8. **添加子目录**
```cmake
add_subdirectory(<name>)
```

9. **添加包含库**
```cmake
add_library(<name> [STATIC | SHARED | MODULE] [EXCLUDE_FROM_ALL] [<source>...])
```

10. **配置和生成文件**
```cmake
configure_file(<input> <output> [@ONLY] [ESCAPE_QUOTES] [NEWLINE_STYLE style])
```
- **主要功能**：
	- **变量替换**：在输入文件中使用 `@VAR@` 或 `${VAR}` 语法定义的变量会被替换为 CMake 中的当前值。
	- **条件编译**
	- **选项控制**
- **常用参数**：
	- `@ONLY`：只替换 `@VAR@` 形式的变量，不替换 `${VAR}` 形式
	- `ESCAPE_QUOTES`：对反斜杠和引号进行转义
	- `NEWLINE_STYLE`：指定换行符风格（UNIX / DOS 等）






## `CMakePresets.json`

`CMakePresets.json` 是 CMake 3.19 及以上版本引入的配置文件，用于预定义和标准化 CMake 的配置选项、构建选项和测试选项。

主要目的是简化 CMake 项目的配置流程，尤其是在多平台、多配置（如 Debug/Release）或多工具链（如 GCC/Clang/MSVC）的场景下，避免重复输入冗长的命令行参数。

**基本结构**：
```json
{
	"version": 3,  // CMake Presets 版本（必须 ≥3）
	"configurePresets": [],  // 配置预设（选择编译器、生成器等）
	"buildPresets": [],      // 构建预设（选择构建类型、并行编译等）
	"testPresets": [],       // 测试预设（可选）
	"vendor": {}             // 扩展字段（可选）
}
```


### 核心作用

1. **统一管理 CMake 配置**：将常用的 `cmake -D...` 参数固化到文件中。
2. **支持多环境预设**：为不同平台（Windows/Linux/macOS）、不同生成器（Ninja/Make/VS）、不同构建类型（Debug/Release）定义独立的配置。
3. **简化团队协作**：确保所有开发者使用相同的构建配置，减少环境差异导致的问题。
4. **与工具集成**：被 VS Code、CLion、Visual Studio 等 IDE 直接支持，实现一键配置。

### 配置 `configurePresets`

定义不同平台的 CMake 配置，
```json
{
	"version": 3,
	"configurePresets": [
		{
			"name": "macos-clang",
			"displayName": "macOS (Clang)",
			"description": "Use Clang on macOS",
			"generator": "Unix Makefiles",
			"binaryDir": "${sourceDir}/build/macos",
			"cacheVariables": {
				"CMAKE_C_COMPILER": "clang",
				"CMAKE_CXX_COMPILER": "clang++",
				"CMAKE_BUILD_TYPE": "Debug"
			},
			"condition": {
				"type": "equals",
				"lhs": "${hostSystemName}",
				"rhs": "Darwin"
			}
		}
	]
}
```
**字段说明**：
- `name`：预设的唯一标识（命令行中使用）
- `displayName`：在 GUI 中显示的名称
- `generator`：CMake 生成器（如 Unix Makefiles、Ninja、Visual Studio 17 2022）
- `binaryDir`：构建目录（${sourceDir} 是项目根目录）
- `cacheVariables`：传递给 CMake 的 -D 变量（如编译器、构建类型）
- `condition`：条件判断（根据系统自动选择）


### 配置 `buildPresets`

定义如何构建项目，
```json
{
	"buildPresets": [
		{
            "name": "macos-debug",
            "configurePreset": "macos-clang",
            "displayName": "Build (macOS Debug)",
            "configuration": "Debug",
            "jobs": 4
        }
	]
}
```
**字段说明**：
- `configurePreset`：关联的 configurePreset 名称
- `configuration`：构建类型（Debug/Release/RelWithDebInfo）
- `jobs`：并行编译任务数（make -j4 / ninja -j8）


### 示例场景

假设原命令行如下：
```bash
cmake -G "Ninja" -DCMAKE_BUILD_TYPE=Debug -B build -DUSE_OPENMP=ON ..
```

对应的 `CMakePresets.json` 文件：
```json
{
	"version": 3,
	"configurePresets": [
		{
			"name": "ninja-debug",
			"displayName": "Ninja Debug",
			"generator": "Ninja",
			"binaryDir": "${sourceDir}/build",
			"cacheVariables": {
				"CMAKE_BUILD_TYPE": "Debug",
				"USE_OPENMP": "ON"
			}
		}
	]
}
```

**生成构建系统**：
```bash
cmake --preset=ninja-debug
```

**构建项目**：
```bash
cmake --build --preset=ninja-debug
```



## 项目结构规划

一个组织良好的多模块 CMake 项目目录结构通常类似这样：
```
Project/
├── CMakeLists.txt              # 根目录的 CMakeLists.txt
├── src/                        # 主源代码目录
│   ├── CMakeLists.txt          # 主源码模块的 CMakeLists.txt
│   ├── module1/                # 模块1目录
│   │   ├── CMakeLists.txt
│   │   ├── include/            # 模块1对外头文件
│   │   │   └── module1/
│   │   │       └── module1.h
│   │   └── src/
│   │       └── module1.cpp
│   └── module2/                # 模块2目录
│       ├── CMakeLists.txt
│       ├── include/
│       │   └── module2/
│       │       └── module2.h
│       └── src/
│           └── module2.cpp
├── tests/                      # 测试模块目录
│   ├── CMakeLists.txt
│   ├── test_module1.cpp
│   └── test_module2.cpp
├── subproject/                 # 子模块目录
│   ├── CMakeLists.txt
│   └── src/
│       └─ subproject_main.cpp
└── build/                      # 构建输出目录（可选）
```

**作用**：
1. **`./CMakeLists.txt`**：项目总入口，设置全局配置、添加子目录（`src`，`tests`，`subproject`）
2. **`./src/CMakeLists.txt`**：定义主程序或核心库，并添加各子模块（`module1`, `module2`）。
3. **`./src/moduleX/CMakeLists.txt`**：定义独立的功能模块，通常编译为库（静态库或动态库）供主程序或其他模块使用。
4. **`./tests/CMakeLists.txt`**：定义组织测试代码，用于验证 `src` 中模块的功能。
5. **`./subproject/CMakeLists.txt`**：定义子项目程序。


### 示例

1. **`./CMakeLists.txt`**：
```cmake
# ./CMakeLists.txt
cmake_minimum_required(VERSION 3.15)  # 指定CMake最低版本要求

project(MyProject VERSION 1.0.0 LANGUAGES CXX C)  # 定义项目名、版本和语言（C++和C）

# 设置C++标准
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# 设置输出目录（可选，但有助于保持构建目录的整洁）
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)   # 静态库输出目录
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)   # 动态库输出目录
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)   # 可执行文件输出目录

# 添加主源代码模块
add_subdirectory(src)

# 条件编译：如果启用测试，则添加tests目录
option(BUILD_TESTS "Build the tests" ON)  # 定义一个选项，默认ON
if(BUILD_TESTS)
    add_subdirectory(tests)
endif()

# 添加工具模块，排除在默认构建目标外（需显式构建）
add_subdirectory(subproject EXCLUDE_FROM_ALL)  # 使用EXCLUDE_FROM_ALL
```


2. **`./src/CMakeLists.txt`**：
```cmake
# ./src/CMakeLists.txt

# 添加项目的主要模块
add_subdirectory(module1)
add_subdirectory(module2)

# 假设主程序入口在src目录下，并依赖module1和module2
file(GLOB_RECURSE MAIN_SOURCES CONFIGURE_DEPENDS "*.cpp" "*.c")  # 递归获取源文件（谨慎使用GLOB*）
# 更好的做法是显式列出源文件：
# set(MAIN_SOURCES main.cpp helper.cpp)

# 创建可执行文件
add_executable(${PROJECT_NAME}_main ${MAIN_SOURCES})  # 例如 MyProject_main

# 为主程序指定头文件搜索路径（如果主程序有自己的头文件目录）
target_include_directories(${PROJECT_NAME}_main PRIVATE include)

# 链接主程序所依赖的模块库
target_link_libraries(${PROJECT_NAME}_main PRIVATE module1 module2)
```


3. **`./src/moduleX/CMakeLists.txt`**：
```cmake
# ./src/moduleX/CMakeLists.txt

# 递归获取模块的源文件和头文件（CONFIGURE_DEPENDS有助于在添加新文件后自动重新运行CMake）
file(GLOB_RECURSE MODULEX_SRCS CONFIGURE_DEPENDS src/*.cpp src/*.c include/*.h)
# 再次强调，显式列出文件是更可靠的做法。

# 创建静态库（也可以选择SHARED创建动态库）
add_library(moduleX STATIC ${MODULE1_SRCS})

# 为这个模块库指定头文件搜索路径。
# 使用PUBLIC，这样任何链接moduleX的目标（如主程序）也会自动添加这个路径。
target_include_directories(moduleX PUBLIC include)

# 如果moduleX还依赖其他库（例如第三方库），可以在这里链接
# target_link_libraries(moduleX PUBLIC SomeThirdPartyLib)
```

4.  **`./tests/CMakeLists.txt`**
```cmake
# ./tests/CMakeLists.txt

# 查找测试框架，例如GTest
find_package(GTest REQUIRED)

# 获取测试源文件
file(GLOB_RECURSE TEST_SOURCES CONFIGURE_DEPENDS *.cpp *.c)

# 创建测试可执行文件
add_executable(tests ${TEST_SOURCES})

# 为测试程序指定头文件搜索路径
target_include_directories(tests PRIVATE ${GTEST_INCLUDE_DIRS}) # GTest的头文件路径
target_include_directories(tests PRIVATE ${CMAKE_SOURCE_DIR}/src/module1/include) # 模块1的头文件
target_include_directories(tests PRIVATE ${CMAKE_SOURCE_DIR}/src/module2/include) # 模块2的头文件

# 链接测试框架和待测试的模块
target_link_libraries(tests ${GTEST_LIBRARIES} module1 module2)

# 可选：添加CTest测试
enable_testing()
add_test(NAME tests COMMAND tests)
```


5. **`./subproject/CMakeLists.txt`**：
```cmake
# ./subproject/CMakeLists.txt

# 获取工具源文件
file(GLOB_RECURSE SUBPROJECT_SOURCES CONFIGURE_DEPENDS *.cpp *.c)

# 创建工具可执行文件
add_executable(subproject ${SUBPROJECT_SOURCES}) # 或者根据文件创建多个工具 add_executable(subproject subproject_main.cpp)

# 指定头文件搜索路径
target_include_directories(subproject PRIVATE ${CMAKE_SOURCE_DIR}/src/module1/include)

# 链接工具所依赖的库
target_link_libraries(subproject PRIVATE module1)
```


## 安装配置

**下载地址**：[Download CMake](https://cmake.org/download/)

当 CMake 安装成功后，你可以选择从命令行或者 GUI 启动 CMake

### 编译流程

#### Windows

Windows 上，打开 CMake 可以看到一个 GUI ，
![](imgs/Pasted%20image%2020240103172645.png)

CMake 需要一个源代码目录和一个存放编译结果的目标文件目录，在设置完源代码目录和目标目录之后，点击 Configure(设置) 按钮，让 CMake 读取设置和源代码。

我们接下来需要选择工程的生成器，由于我们使用的是 Visual Studio 2019，我们选择 Visual Studio 16 选项（因为Visual Studio 2019的内部版本号是16）。CMake会显示可选的编译选项用来配置最终生成的库。这里我们使用默认设置，并再次点击Configure(设置)按钮保存设置。保存之后，点击Generate(生成)按钮，生成的工程文件会在你的build文件夹中。

在build文件夹里可以找到 `*.sln` 文件。

#### Linux

Linux 平台下使用 CMake 生成 Makefile 并编译的流程如下：
1. 写 CMake 配置文件 `CMakeLists.txt` 。
2. 执行命令 `cmake [PATH]` 或者 `ccmake [PATH]` 生成 Makefile。
3. 使用 `make` 命令进行编译。




## CMake Converter

将其他项目文件转换为CMake：[Use CMake Converter — CMake-Converter 2.1 documentation](https://cmakeconverter.readthedocs.io/en/latest/use.html)

