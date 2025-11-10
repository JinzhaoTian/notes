vcpkg 是微软推出的一个开源 C/C++ 包管理工具，用于帮助开发者在不同平台上（Windows、Linux、macOS）快速获取、编译和管理第三方库依赖。它简化了 C/C++ 项目中库的安装和集成过程，支持跨平台开发，并与 Visual Studio、CMake 等工具链深度集成。

## 核心特点

1. **跨平台支持**
    - 支持 Windows（MSVC/MSBuild/MinGW）、Linux 和 macOS（Clang/GCC）。
    - 提供一致的命令行接口，操作方式统一。

2. **自动化依赖管理**
    - 自动下载库的源代码并编译，解决头文件和库文件的路径问题。
    - 支持版本控制（如 `vcpkg.json` 声明依赖版本）。

3. **与构建工具集成**
    - **Visual Studio**：直接识别 `vcpkg` 安装的库，无需手动配置。
    - **CMake**：通过 `find_package` 或工具链文件（`vcpkg.cmake`）集成。
    - **其他工具链**：支持 Meson、Makefile 等。

4. **庞大的库生态**
    - 提供超过 **2000+** 个开源库（如 Boost、OpenCV、SDL2、fmt 等），通过社区维护的“端口（ports）”扩展。

5. **自定义编译选项**
    - 支持静态库/动态库、Debug/Release 编译，可指定目标架构（x86、x64、ARM 等）。



## 核心概念

### triplet

vcpkg 的 triplet（三元组）是一个核心概念，它定义了构建库时的目标平台配置。一个 triplet 通常由三部分组成：
- 目标架构（如 x86，x64，arm）
- 目标平台（如 Windows，Linux，OSX）
- 链接方式（动态链接或静态链接）

#### 常见类型

Triplet 文件位于 vcpkg 安装目录的 `triplets` 子目录中：
- 内置 triplet：`vcpkg/triplets/`
	- `x64-windows-static`：静态链接
	- `x64-windows`：动态链接，同时编译 debug 版和 release 版
	- `x64-windows-release`：动态链接，只编译 release 版
	- `x64-linux`：
	- `x64-osx`：动态链接
	- `arm64-osx`：动态链接
- 社区 triplet：`vcpkg/triplets/community/`
	- `x64-mingw-dynamic`
	- `x64-mingw-static`
	- `x64-linux-dynamic`
	- `x64-linux-release`
	- `arm64-osx-dynamic`：动态链接
	- `arm64-osx-release`：动态链接

#### 设置问题

有些预设的 triplet 会同时附带一个 release 版本（如 `x64-windows` 和 `x64-windows-release` ），其二者的核心区别在于**它们构建的库的类型（调试版还是发布版）以及这些库的依赖关系**。

以  `x64-windows` 和 `x64-windows-release` 为例：
1.  **`x64-windows`**：
	- **构建**：会**同时构建调试（Debug）和发布（Release）两种版本的库**
	- **用途**：主要用于开发环境，当切换项目的配置（例如从 Debug 切换到 Release）时，链接器会自动找到对应版本的库文件进行链接，非常方便。
	- **特点**：占用磁盘空间更大，因为它包含了同一库的两个版本。

2. **`x64-windows-release`**：
	- **构建**：**仅构建发布（Release）版本的库**
	- **用途**：主要用于生产环境、持续集成（CI）流水线或最终发布。在这些场景下，只需要生成最终的、优化的可执行文件，不需要调试版的库
	- **特点**：更轻量、更高效，但**不适合日常开发调试**。
		- 显著减少安装时间和磁盘空间占用（大约节省一半）。

> [!tip] 使用建议
> **在开发调试阶段时**，建议使用 `x64-windows` 这类会同时构建 Debug 和 Release 版本的 triplet 配置。vcpkg 的 CMake 工具链能自动根据项目生成配置（CMake 的 Debug 或 Release）选择链接对应版本的库，省去手动配置的麻烦。
> 
> **在发布阶段或 CI 时**，建议使用 `x64-windows-release` 这类仅构建 Release 版本的 triplet 配置，这样会有更小的体积和更高的性能，并**确保项目生成配置也为 Release 模式**。

> [!warning] 在开发调试阶段，不要混用 Release 版的第三方库
> 因为 Debug 和 Release 版本在一些关键设置上存在差异，所以可能会发生一些问题如：
> 1. **运行时库（CRT）不匹配**：（**最常见的问题**）
> 	- 如在 Windows 下，Debug版本的程序会链接 Debug 版本的运行时库（如 `MDd`），而 Release 版本的程序链接 Release 版本的运行时库（如 `MD`）。
> 	- 如果你的程序是 Debug 版本，却尝试链接一个 Release 版本的第三方库，而这个库又期望链接 Release 版本的运行时库，就会导致链接冲突或运行时错误。
> 2. **迭代器调试级别不匹配**：
> 	- 如 `MSVC` 的 STL 中的迭代器在 Debug 和 Release 模式下有不同的安全检查级别（通过 `_ITERATOR_DEBUG_LEVEL` 宏控制），Debug 版本中该值更高以检测更多错误，混合模式会导致运行时关于迭代器的断言失败或崩溃。
> 3. **调试信息缺失**：
> 	- Release 版本的库通常被优化并且**不包含调试符号**，这意味着当你的代码调用这些库时，调试器无法深入到库的代码中进行单步调试，也无法在调用栈中清晰显示库内部的函数名，使得诊断问题变得困难。
> 4. **优化带来的调试困扰**：
> 	- Release 版本的库编译器会进行**激进优化**，这可能使得调试时变量值无法查看、函数调用流程难以跟踪，导致调试体验很差。


## 命令


1. **安装**
```bash
vcpkg install
```

2. **更新与卸载**
```bash
vcpkg update      # 查看可更新的库
vcpkg upgrade     # 更新所有库
vcpkg remove zlib # 卸载库
```



## 使用示例


### 安装 vcpkg

1. **下载安装**：
```bash
git clone https://github.com/microsoft/vcpkg
./vcpkg/bootstrap-vcpkg.sh  # Linux/macOS
.\vcpkg\bootstrap-vcpkg.bat # Windows
```

2. **添加 vcpkg 的 PATH 设置**


### 集成到项目

1. **项目结构**
```
Project/
├── CMakeLists.txt
├── vcpkg.json         # 可选（Manifest 模式）
├── src/
│   ├── main.cpp
│   └── ...
└── build/
```

2. **配置 `vcpkg.json`**（可选，推荐使用 Manifest 模式）：
```json
{
    "name": "jzre",
    "version": "1.0.0",
    "dependencies": [
        {
            "name": "glfw3",
            "version>=": "3.4#1"
        },
        {
            "name": "glad",
            "version>=": "0.1.36"
        },
        {
            "name": "imgui",
            "version>=": "1.91.9",
            "features": ["glfw-binding", "opengl3-binding", "docking-experimental"]
        }
    ],
    "builtin-baseline": "5d8a81e7e232ca70c62f645b0ff434ac5574e921"
}
```

3. **安装依赖**：
```bash
vcpkg install
```

### 配置 CMake

1. **配置 `CMakeLists.txt`**：
```cmake
cmake_minimum_required(VERSION 3.20)

project(JzRE)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# find packages using vcpkg
find_package(glfw3 CONFIG REQUIRED)
find_package(glad CONFIG REQUIRED)
find_package(imgui CONFIG REQUIRED)

# add platform-specific libraries and OpenGL
if(WIN32)
    find_package(OpenGL REQUIRED)
    set(PLATFORM_LIBS opengl32 gdi32)
    set(OPENGL_LIBS ${OpenGL})
elseif(APPLE)
    # On macOS, OpenGL is part of the system frameworks
    find_library(COCOA_LIB Cocoa)
    find_library(IOKIT_LIB IOKit)
    find_library(COREVIDEO_LIB CoreVideo)
    find_library(OPENGL_LIB OpenGL)
    find_library(GLUT_LIB GLUT)
    find_library(UNIFORMTYPEIDENTIFIERS_LIB UniformTypeIdentifiers)
    set(PLATFORM_LIBS ${COCOA_LIB} ${IOKIT_LIB} ${COREVIDEO_LIB} ${UNIFORMTYPEIDENTIFIERS_LIB})
    set(OPENGL_LIBS ${OPENGL_LIB} ${GLUT_LIB})
elseif(UNIX AND NOT APPLE)
    find_package(OpenGL REQUIRED)
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(X11 REQUIRED x11)
    set(PLATFORM_LIBS ${X11_LIBRARIES} ${CMAKE_DL_LIBS})
    set(OPENGL_LIBS ${OpenGL})
endif()

# set project include path
include_directories(${PROJECT_SOURCE_DIR}/src)

# set project source files
file(GLOB JZRE_SOURCES "${PROJECT_SOURCE_DIR}/src/*.cpp")

# combine all sources
set(ALL_SOURCES ${JZRE_SOURCES} ${PLATFORM_SOURCES})

# add exe target
add_executable(JzRE "${PROJECT_SOURCE_DIR}/src/main.cpp" ${ALL_SOURCES})

# link libraries using vcpkg targets
target_link_libraries(JzRE PRIVATE
    ${PLATFORM_LIBS}
    ${OPENGL_LIBS}
    glad::glad
    imgui::imgui
)
```


2. **配置 `CMakePresets.txt`**：
```cmake
{
    "version": 3,
    "configurePresets": [
        {
            "name": "linux-gcc",
            "displayName": "linux-gcc",
            "description": "Use GCC on Linux",
            "generator": "Unix Makefiles",
            "binaryDir": "${sourceDir}/build",
            "cacheVariables": {
                "CMAKE_C_COMPILER": "gcc",
                "CMAKE_CXX_COMPILER": "g++",
                "CMAKE_BUILD_TYPE": "Debug",
                "CMAKE_TOOLCHAIN_FILE": "${env.VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake",
                "VCPKG_TARGET_TRIPLET": "x64-linux",
                "VCPKG_HOST_TRIPLET": "x64-linux"
            },
            "condition": {
                "type": "equals",
                "lhs": "${hostSystemName}",
                "rhs": "Linux"
            }
        },
        {
            "name": "macos-clang",
            "displayName": "macos-clang",
            "description": "Use Clang on macOS",
            "generator": "Unix Makefiles",
            "binaryDir": "${sourceDir}/build",
            "cacheVariables": {
                "CMAKE_C_COMPILER": "clang",
                "CMAKE_CXX_COMPILER": "clang++",
                "CMAKE_BUILD_TYPE": "Debug",
                "CMAKE_TOOLCHAIN_FILE": "${env.HOME}/vcpkg/scripts/buildsystems/vcpkg.cmake",
                "VCPKG_TARGET_TRIPLET": "arm64-osx",
                "VCPKG_HOST_TRIPLET": "arm64-osx"
            },
            "condition": {
                "type": "equals",
                "lhs": "${hostSystemName}",
                "rhs": "Darwin"
            }
        },
        {
            "name": "windows-msvc",
            "displayName": "windows-msvc",
            "description": "Windows using MSVC compiler with dynamic linking",
            "generator": "Ninja",
            "binaryDir": "${sourceDir}/build",
            "cacheVariables": {
                "CMAKE_C_COMPILER": "cl",
                "CMAKE_CXX_COMPILER": "cl",
                "CMAKE_BUILD_TYPE": "Debug",
                "CMAKE_TOOLCHAIN_FILE": "${env.VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake",
                "VCPKG_TARGET_TRIPLET": "x64-windows",
                "VCPKG_HOST_TRIPLET": "x64-windows",
                "CMAKE_MSVC_RUNTIME_LIBRARY": "MultiThreadedDebugDLL"
            },
            "environment": {
                "CFLAGS": "/utf-8",
                "CXXFLAGS": "/utf-8 /EHsc"
            },
            "condition": {
                "type": "equals",
                "lhs": "${hostSystemName}",
                "rhs": "Windows"
            }
        }
    ]
}

```

### 构建与运行

```bash
cmake -B build -S .
cmake --build build
```


### 配置 Visual Studio


1. **全局集成**：在 vcpkg 目录下，以管理员身份运行执行以下命令：
```powershell
.\vcpkg integrate install
```
会修改 Visual Studio 的系统级配置，成功后会显示：`Applied user-wide integration for this vcpkg root.`，此后在 Visual Studio 中创建一个新的 C++ 项时，可以直接包含 vcpkg 的库。

2. **项目级集成**：Visual Studio 对 CMake 有原生支持。
	- 在 Visual Studio 的解决方案资源管理器中，右键点击 `CMakeLists.txt`，选择“CMake Settings for Project”。
	- 在打开的界面中，点击“Edit JSON”，直接在 `CMakeSettings.json` 中添加 `"VCPKG_TOOLCHAIN"` 配置。
	- 在 `configurations` 数组中的每个你需要用的配置里（如 `x64-Debug`），添加以下两行（请修改为你的实际 vcpkg 路径）：
```json
{
  "configurations": [
    {
      "name": "x64-Debug",
      "generator": "Ninja",
      "configurationType": "Debug",
      "buildRoot": "${projectDir}\\out\\build\\${name}",
      "installRoot": "${projectDir}\\out\\install\\${name}",
      "cmakeCommandArgs": "",
      "buildCommandArgs": "",
      "ctestCommandArgs": "",
      "variables": [],
      // 以下是关键配置 ↓↓↓
      "cmakeToolchain": "C:/dev/vcpkg/scripts/buildsystems/vcpkg.cmake",
      "variables": [
        {
          "name": "VCPKG_TARGET_TRIPLET",
          "value": "x64-windows"
        }
      ]
    }
  ]
}
```


