DirectX 12 Agility SDK 是为 DirectX 12 开发准备的最新 SDK，采用“即用即取”的模式，核心文件会随着应用一起分发，因此安装方式也与传统 SDK 不同。


## 通过 Visual Studio 安装

1. **安装**：
    - 在 Visual Studio 中打开你的项目。
    - 在“解决方案资源管理器”中，右键点击项目名称，选择“**管理 NuGet 程序包**”。
    - 在打开的界面中，点击“浏览”选项卡，然后在搜索框中输入“**DirectX 12 Agility**”。
    - 在搜索结果中找到 `Microsoft.Direct3D.D3D12` 包，选择最新稳定版本，然后点击右侧的“**安装**”即可。
2. **验证**：安装成功后，你会在项目输出目录（例如 `x64/Debug`）下看到一个名为 `D3D12` 的文件夹，其中包含了 `D3D12Core.dll` 等核心组件。


## 在 CMake 中集成

### 方法 A：使用 vcpkg（推荐）

`vcpkg` 是微软维护的 C++ 包管理器，可以非常方便地安装和管理 DirectX 相关的库。

1. **安装 vcpkg**：如果你还没有安装 vcpkg，可以先从 GitHub 克隆并运行引导程序。
2. **安装必要的包**：在 vcpkg 的目录下，运行以下命令来安装 Agility SDK 相关的包。`directx-headers` 包含了官方的 Direct3D 12 头文件，`directx-dxc` 是 DirectX 着色器编译器。
```bash
# 为 x64 架构安装
.\vcpkg install directx-headers directx-dxc --triplet x64-windows
```

3. **在 CMake 中集成**：你需要通过 CMake 工具链文件将 vcpkg 集成到你的项目中。之后，就可以使用 `find_package` 来查找并使用这些库。
```cmake
# CMakeLists.txt

cmake_minimum_required(VERSION 3.10)
project(MyD3D12Project)

# 查找 vcpkg 提供的包
# 这些包会导入对应的 CMake 目标 (targets)
find_package(directx-headers CONFIG REQUIRED)
find_package(directx-dxc CONFIG REQUIRED)

# 添加你的可执行文件
add_executable(MyApp main.cpp)

# 链接库到你的项目
# 链接到 directx-headers 提供的目标，它会自动设置包含目录
# 链接到 directx-dxc 提供的编译器库目标 [citation:3][citation:8]
target_link_libraries(MyApp PRIVATE
    Microsoft::DirectX-Headers
    Microsoft::DirectXShaderCompiler
)

# 如果你还需要 DirectX Tool Kit 等辅助库，也可以用类似方式安装和链接 [citation:6]
# find_package(directxtk12 CONFIG REQUIRED)
# target_link_libraries(MyApp PRIVATE Microsoft::DirectXTK12)
```


