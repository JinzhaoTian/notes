
## macOS

1. 第一步：**下载并安装 SDK**
	- **获取安装包**：访问 LunarG 官方网站的 [Vulkan SDK 下载页面](https://vulkan.lunarg.com/sdk/home)，选择适用于 macOS 的版本进行下载。
	- **运行安装器**：打开下载好的 `.dmg` 文件，运行其中的安装程序。安装的核心组件包括：
	    - **Vulkan Loader**：你的应用程序与系统图形驱动（这里是 MoltenVK）之间的桥梁。
	    - **验证层（Validation Layers）**：能在你编写和调试程序时，提供详细的错误检查和诊断信息，帮助你避免许多常见的 Vulkan 使用错误。
	    - **MoltenVK**：这是让 Vulkan 在 macOS 上运行的关键组件。它作为一个“运行时”，将 Vulkan API 的命令实时“翻译”成 macOS 的原生图形 API —— Metal 能够理解的语言。
	    - **工具集**：包括用于查看设备信息的 `vulkaninfo`、测试用的 `vkcube`，以及用于将着色器编译为 SPIR-V 字节码的 `glslc` 等工具。

2. 第二步：**验证安装**
	- 打开 Terminal
	- 运行测试程序 `vkcube`。如果屏幕上出现一个旋转的彩色立方体，说明你的Vulkan环境（包括MoltenVK转换层）已经成功搭建，显卡驱动也兼容支持。
	- 你也可以运行 `vulkaninfo` 命令，它会打印出关于你的 Vulkan 驱动、支持的功能和设备的详细信息。
```bash
/Applications/vulkan/bin/vkcube
```


3. 第三步：**配置开发环境**，为了让你的编译器和构建系统能找到 Vulkan SDK，需要配置环境变量。SDK 通常会提供一个脚本帮你自动完成。
	- **设置环境变量**：在终端中，找到并运行 SDK 自带的环境设置脚本。脚本的路径类似于 `~/VulkanSDK/[版本号]/macOS/setup-env.sh`。运行后，它会自动设置好 `VULKAN_SDK`、`PATH`、`DYLD_LIBRARY_PATH` 等关键变量。
	- **在项目中使用**：在你的 C++ 项目中，需要配置包含目录和链接库，才能使用 Vulkan 进行开发。
	    - **包含目录**：将 `$VULKAN_SDK/include` 添加到编译器的头文件搜索路径中。
	    - **链接库**：将 `$VULKAN_SDK/lib` 添加到库搜索路径，并链接 `vulkan` 库。
	    - **使用CMake**：如果你的项目使用CMake构建，可以方便地通过 `find_package(Vulkan REQUIRED)`命令来自动处理这些链接和包含路径的设置。

### 额外依赖

为了方便开发，你可能还需要安装一些常用的辅助库，例如窗口管理库GLFW和数学库GLM。如果你安装了Homebrew，可以通过以下命令快速完成[](https://eazydevelop-community.eazytec-cloud.com/6912cd6e5511483559e6edaa.html)[](http://docs.ros.org/en/ros2_packages/jazzy/api/librealsense2/doc/installation_osx.html)：

```bash
brew install glfw glm
```

### 关键点

- **macOS 上的 Vulkan ≠ 原生 Vulkan**：请务必理解，macOS 上的 Vulkan 是通过 MoltenVK 转换层实现的，它并非显卡硬件的原生驱动。因此，它可能不支持完整的 Vulkan 1.3 功能集，并且在某些极致的性能场景下，可能与原生 Vulkan 或 Metal 略有差异。
- **Xcode 是必须的**：Vulkan SDK 依赖于苹果的开发者工具，确保你已经安装了最新版的 **Xcode** 和 **Xcode命令行工具**。
