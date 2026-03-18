Godot Engine is a feature-packed, cross-platform game engine to create 2D and 3D games from a unified interface. It provides a comprehensive set of common tools, so that users can focus on making games without having to reinvent the wheel. Games can be exported with one click to a number of platforms, including the major desktop platforms (Linux, macOS, Windows), mobile platforms (Android, iOS), as well as Web-based platforms and consoles.

Godot 是一个通用型的 2D 和 3D 游戏引擎，最初由阿根廷的一家游戏工作室于 2001 年开发，并于 2014 年开源，采用 MIT 许可证。

主要支持两种官方脚本语言：
- **GDScript**：Godot 专属的语言，语法很像 Python，专门为 Godot 设计，特点是简单易学，与引擎集成度极高。
- **C#**：如果你有 Unity 或其他 .NET 开发经验，可以直接使用 C# 进行开发，性能表现优秀。此外，通过 GDExtension (Godot 4.x) 技术，你还可以使用 C++ 等语言编写高性能模块。

Godot 的核心设计理念是场景和节点，游戏中的所有东西（比如一个角色、一个按钮、一束光）都是一个节点，而多个节点有层次地组合起来，就构成了一个场景（比如一个游戏关卡或一个玩家角色），这种设计让游戏逻辑的组织变得非常清晰和直观。

Godot 提供了一个功能齐全的编辑器，里面集成了开发游戏所需的绝大多数工具，比如动画编辑器、 tilemap 编辑器（用于快速搭建 2D 地图）、着色器编辑器、调试器、性能分析工具等，你无需再寻找和配置一大堆外部软件。


Godot 在 2D 游戏开发方面非常出色，它的 3D 能力也在通过最新的 Vulkan 渲染器等技术的支持下不断增强。

## Godot 架构

这种自举的能力，得益于 Godot 清晰的代码分层：
- **`editor/`**：编辑器的专属代码，就像你的游戏项目代码。它“使用”下面的所有模块。
- **`scene/`**：游戏和编辑器公用的所有 2D / 3D 节点、UI 控件。编辑器也是靠这些控件搭建的。
- **`servers/`**：底层服务，如渲染器、音频、物理引擎。它们负责把逻辑变成屏幕上的像素和声音。
- **`core/`**：最核心的类、数据结构等，是引擎的基石


## Godot 编辑器

Godot 编辑器本身就是一个“自托管”的应用程序。它并不是用外部工具包（比如 Qt 或 GTK ）做的，而是完全构建在它自己的引擎核心之上，就像它运行的一个特殊“游戏项目”。

### 技术实现

在技术实现上，当你从源码编译一个编辑器版本的 Godot 时，它的启动流程很特殊：
- **启动入口**：程序会首先加载并执行一段特殊的 C++ 代码，这个代码文件在源码里叫 `editor/editor_node.cpp`，你可以把它想象成编辑器这个“游戏”的 **“主场景”** 。
- **构建 UI**：这段 C++ 代码会像你在游戏脚本里 `instance` 一个场景一样，去调用引擎底层的渲染器和 UI 系统，把编辑器里的每一个按钮、每一个面板、每一个视口都**动态地创建和绘制出来**。
- **核心依赖**：为了保持代码清晰，编辑器的代码（`editor/`）可以调用更底层的场景、服务、核心模块。但反过来，底层的模块则不能依赖编辑器，这就保证了即便没有编辑器，引擎的核心部分也能独立运行你的游戏。

简单来说，整个编辑器界面，在运行时就是一个由 C++ 代码动态构建起来的、巨大的、可交互的 “场景树”。它的存在，依赖于引擎最底层的渲染、输入、GUI 等所有服务。



[Godot 4.x GDScript Best Practices - Coding Standards | SkillKit | SkillKit](https://skillkit.io/zh/skills/claude-code/godot-best-practices)