提到游戏开发，一定会想到 C/C++、DirectX、OpenGL 等这些东西，但在开发时如果采用编译型的语言，当更新一些逻辑时，整个项目都需要重新编译，大大地影响开发效率。

现在游戏界普遍采用的方式是将游戏的底层逻辑交给 C/C++ 这样的底层语言，而将游戏的上层逻辑交给脚本语言。例如，Unreal Engine 是采用 [UnrealScript](../Unreal%20Engine/UE%20UnrealScript.md) ；Unity3D 是采用的 C#、javaScript等；cocos2d-x 采用 Lua、javaScript 等。

常见的用于游戏开发的脚本语言：
- [Lua](../../Language/Lua/Lua.md) 
- [C#](../../Language/CS/CS.md)
- [JavaScript](../../Language/JavaScript/JavaScript.md)


## 集成

### 脚本引擎

- **Lua** ：可以使用 `LuaBridge` 或 `Sol2` 等库来绑定 Lua 与 C++ 之间的接口。
- **C#** ：可以通过 Mono 或 .NET 集成。
- **Python** ：可以通过 `pybind11` 或 `Boost.Python` 等工具实现绑定。
- **JavaScript** ：可以集成 V8 或 QuickJS 等 JavaScript 引擎。

### 脚本绑定

为了让脚本能够控制游戏引擎的各种功能，需要将引擎 API 暴露给脚本。绑定的方式取决于选择的脚本语言和引擎的架构。
- **暴露C++类和函数**：将引擎中的关键功能（如游戏对象、渲染、物理、输入等）通过 C++ 代码暴露给脚本。您可以使用自动生成的绑定代码或手动编写绑定代码。例如，使用 `LuaBridge` 或 `Sol2` 库来将 C++ 类和函数暴露给 Lua 。
- **对象生命周期管理**：在绑定过程中，需要确保对象的生命周期管理正确。例如，Lua 和 C++ 之间的对象共享时，必须管理好对象的创建和销毁，避免内存泄漏或非法访问。
- **事件和回调**：脚本系统常常通过事件或回调机制与游戏引擎的其他模块进行交互。例如，游戏引擎中有一个定时器功能，当定时器触发时，可以在脚本中注册一个回调函数，响应事件。

### 脚本加载

在游戏运行时，脚本可以从文件加载。您可以设计脚本管理系统，支持从文件、网络或内存中加载脚本。
- 对于 Lua ，使用 `luaL_dofile` 或 `luaL_loadfile` 来加载脚本文件。
- 对于 Python ，使用 `PyRun_SimpleString` 或 `execfile` 来加载 Python 脚本。

**优化**：脚本热加载，支持在运行时重新加载脚本，而不需要重启游戏。可以使用文件监听来实现热加载功能。


### 脚本执行

通过脚本引擎提供的 API 执行脚本。对于 Lua ，可以通过 `lua_pcall` 执行代码；对于 Python ，可以使用 `PyRun` 等函数。

**优化**：一些语言（如 Lua ）有 JIT（即时编译）支持，可以通过开启 JIT 提高脚本执行性能。


## C# 与 Mono

将 Mono 集成到游戏引擎的脚本系统中可以使开发者在游戏开发中使用 C# 作为脚本语言。通过集成 Mono ，游戏引擎可以让开发者编写、执行和调试 C# 脚本，以便更灵活地控制游戏逻辑、组件和事件。集成 Mono 到游戏引擎的过程通常包括以下几个步骤：

### 1. 引入 Mono 运行时

首先，你需要将 Mono 运行时集成到游戏引擎中。Mono 是跨平台的，因此你可以使用 Mono 在多个操作系统上运行你的游戏引擎。你可以通过以下方式将 Mono 集成到引擎中：

- 下载并编译 Mono 运行时。
- 将 Mono 的核心库（如 `mono.dll` 或 `libmono.so` 等）和头文件集成到引擎的构建系统中。
- 确保运行时与引擎代码的调用和内存管理兼容，特别是考虑到 C# 和 C++ 之间的内存管理差异。

### 2. 设置 Mono 执行环境

在游戏引擎中，Mono 需要运行时环境来执行 C# 代码。这通常包括：

- **编译和加载程序集（Assembly）**：引擎需要能够将 C# 代码编译成中间语言（IL），然后将其加载为 Mono 的程序集（ `.dll` 文件）。你可以通过 Mono 的 API 加载这些程序集。
```cpp
MonoDomain* domain = mono_jit_init("GameDomain");
MonoAssembly* assembly = mono_domain_assembly_open(domain, "MyScript.dll");
```
- **创建Mono域（Mono Domain）**：Mono 运行时通过 `MonoDomain` 管理应用程序的所有执行上下文。每个游戏实例可以有自己的 `MonoDomain` ，以便在不同的上下文中运行不同的脚本。


### 3. 绑定游戏引擎的功能

要使 C# 脚本能够操作游戏引擎中的对象和系统，你需要将游戏引擎的 API 暴露给 Mono。这通常通过**绑定**或**托管代码**的方式完成，允许 C# 脚本与 C++ 引擎之间进行交互。

- **P/Invoke（平台调用）**：使用 C++ 的 API ，Mono 可以通过 `P/Invoke` 机制来调用本地 C++ 函数或方法。你可以将 C++ 函数声明为 C# 可调用的方式。
```cpp
extern "C" void my_native_function() {
    // C++ function
}
```

```C#
[DllImport("MyGameEngine.dll")]
public static extern void my_native_function();
```


- **Mono Embedding**：为了与 C# 脚本交互，游戏引擎通常会使用 Mono 的 API ，将引擎对象和 C# 脚本中的类、方法绑定在一起。这通常是通过注册引擎中的 C++ 类和 C# 类的映射来完成的，允许 C# 脚本调用引擎中的类和方法。
```cpp
MonoClass* class = mono_class_from_name(domain, "MyGameEngineNamespace", "GameObject");
MonoObject* obj = mono_object_new(domain, class);
```


### 4. 执行和调用 C# 脚本

一旦 Mono 集成到引擎中并绑定了 API ，你就可以通过 Mono 运行时执行 C# 脚本了。可以动态加载 C# 脚本并调用脚本中的方法。

- **调用 C# 方法**：
```cpp
MonoMethod* method = mono_class_get_method_from_name(class, "Start", 0);
mono_runtime_invoke(method, obj, nullptr, nullptr);
```
这段代码会调用 C# 类 `GameObject` 的 `Start` 方法。你可以在游戏引擎中执行这些方法来控制游戏行为。


### 5. 内存管理和垃圾回收

C# 的内存管理由 Mono 的垃圾回收（GC）机制控制，因此你需要确保游戏引擎中的对象与 Mono 中的对象能够正确协作。为了避免内存泄漏或错误的内存访问，你需要正确管理 C++ 和 C# 之间的内存边界。

- 引擎应负责在 C# 脚本结束时清理 C++ 对象。
- 使用 `GCHandle` 来保持对 C# 对象的引用，防止它们被垃圾回收器错误地回收。

### 6. 调试与热重载

许多游戏引擎支持热重载（Hot Reload）功能，允许开发者在运行时修改脚本并立即看到结果。通过集成 Mono ，开发者可以在游戏运行时加载、重新加载或替换 C# 脚本，以便进行快速迭代。

### 7. **性能优化**

将 Mono 集成到游戏引擎中时，性能可能是一个挑战，因为 C# 脚本和 C++ 代码之间的交互可能带来开销。为了优化性能，可以：

- 缓存 Mono 对象和方法，避免每次调用时的额外开销。
- 使用对象池来管理频繁创建和销毁的 Mono 对象。
- 在 C# 脚本中避免频繁调用性能敏感的 C++ 函数，尽量减少跨语言的边界。


### [IL2CPP](../Unity/IL2CPP.md)

IL2CPP（Intermediate Language to C++）是 Unity 引擎使用的一种技术，它在将 C# 代码转换为 C++ 代码后进行编译，从而提高性能并生成适用于不同平台的原生代码。

IL2CPP 在 Mono 与引擎的交互中起到了桥梁作用，尤其是在 C++ 引擎层与托管代码层之间。尽管 Mono 也能通过 C# 运行时执行代码，IL2CPP 的存在使得游戏引擎能够：
- 直接访问 C++ 代码：通过将 C# 代码转为 C++，引擎可以更方便地与底层 C++ 库交互。
- 内存管理：IL2CPP 允许更精细的内存控制，可以在 C++ 中控制对象生命周期，从而使得内存使用更加高效。

在 Unity 中，**Mono 和 IL2CPP 并不是互相排斥的技术，而是用于不同的目的**。具体而言：
- Mono：是运行时执行托管代码的核心工具。在 Unity 编辑器中，Mono 用于**在开发时**运行和调试 C# 代码。Mono 在运行时提供了对 C# 代码的支持，允许开发者实时测试和修改代码。
- IL2CPP：是一个编译时工具，**在游戏打包时**发挥作用。IL2CPP 将 C# 代码转换为 C++ 并生成原生代码，这对于最终的游戏发布至关重要。它并不直接在运行时与 Mono 交互，而是在打包时将 C# 代码转化为本地 C++ 代码。



### 游戏打包

在游戏打包过程中，Mono 的作用是将 C# 脚本编译成中间语言（IL），并将其嵌入到最终的游戏包中。

具体步骤如下：
- **脚本编译**：Mono 编译器（如 `mcs` 或 `csc` ）将 C# 脚本编译成 .NET 的中间语言（IL），通常保存在一个或多个 `.dll` 文件中。
- **程序集嵌入**：打包过程中，编译后的 `.dll` 文件（包含游戏脚本）被嵌入到游戏的资源包或安装包中。游戏引擎会将这些程序集作为数据文件包含进最终的游戏安装包或应用程序中。
  例如，在 Unity 中，所有 C# 脚本（即 `.dll` 文件）都会被打包进一个 `.apk` 文件（ Android 平台）或 `.exe` 文件（ Windows 平台）中。
- **资源管理**：脚本可能会与其他游戏资源一起打包（如纹理、模型等），并根据平台和设备的需要进行优化和压缩。

在游戏启动时，Mono Runtime 会负责加载并执行这些脚本。

具体流程如下：
- **加载 Mono 运行时**：当游戏启动时，Mono Runtime 会被初始化，创建一个 Mono 域（ `MonoDomain` ），并通过该域来管理和执行 C# 脚本，确保所有的 C# 代码都能在这个执行环境下运行。Mono 域是 Mono 执行环境的基础，游戏引擎通过它加载和运行 C# 程序集。
```cpp
MonoDomain* domain = mono_jit_init("GameDomain");
```

- **加载程序集**：游戏引擎通过 Mono Runtime 加载之前打包好的程序集（ `.dll` 文件），并将它们加载到 Mono域中。这些 `.dll` 文件通常会在游戏启动时通过指定的路径或内存加载，如果这些文件被压缩或加密，游戏引擎可能需要解压或解密它们。
```cpp
MonoAssembly* assembly = mono_domain_assembly_open(domain, "GameScripts.dll");
```

- **执行脚本**：加载完程序集后，游戏引擎可以通过 Mono Runtime 的 API 来执行 C# 代码。这通常包括调用脚本中的特定方法或类来控制游戏行为。
  例如，游戏引擎会调用 C# 脚本中的 `Start` 方法（ Unity 中的 `Awake` 或 `Start` ），初始化游戏对象并执行游戏逻辑。
  游戏通常是事件驱动的，Mono Runtime 会根据游戏逻辑和用户输入的事件，在正确的时间调用脚本中的方法。这些方法执行后，可能会更新游戏状态、生成新的物体或响应玩家操作。
```cpp
MonoMethod* startMethod = mono_class_get_method_from_name(class, "Start", 0);
mono_runtime_invoke(startMethod, obj, nullptr, nullptr);
```

- **托管与本地代码交互**：尽管脚本是用 C# 编写的，它们依然需要与底层引擎的本地 C++ 代码进行交互。这通常通过 Mono 的 P/Invoke（平台调用）机制或者引擎专用的绑定代码来实现。
	- **绑定函数**：引擎会定义一些 C++ 函数或类，并通过 Mono API 将这些功能暴露给 C# 脚本。例如，游戏中的物理引擎、图形渲染、用户输入等都可以通过绑定的方式在 C# 脚本中调用。
	- **调用本地代码**：脚本可能会调用引擎提供的本地代码来处理图形渲染、物理计算、音效播放等操作。

- **脚本生命周期管理**：脚本的生命周期管理包括如何初始化、执行、暂停、销毁等。在 Mono 集成后，游戏脚本生命周期的管理通常由引擎控制，以下是一些常见的管理方式：
	- **初始化**：在游戏启动时，Mono 会初始化并加载 C# 脚本。
	- **更新**：游戏引擎的主循环会定期调用 Mono 运行时，执行 C# 脚本中的更新方法（例如， `Update` ）。这些方法会在每一帧中执行，用于处理逻辑、动画等。
	- **卸载与销毁**：在游戏对象销毁时，Mono也会负责清理相关的C#对象，以避免内存泄漏。游戏对象的生命周期结束时，Mono会调用`OnDestroy`方法或类似方法，确保脚本资源被清理。


## C# 和 .NET Core

.NET Core 相对于 Mono 在性能上有优势，特别适合需要高性能的游戏。


## Lua

在游戏引擎中集成 Lua 脚本，通常的步骤如下：

### 1. 选择 Lua 绑定库

要在 C++ 游戏引擎中集成 Lua 脚本，最常见的方式是使用 Lua 绑定库。常用的 Lua 绑定库有：
- **LuaBridge**：一个 C++ 到 Lua 的绑定库，易于使用且高效。
- **Sol2**：现代 C++ 绑定库，提供了更加简洁和强大的API。
- **luabind**：较老的绑定库，提供了 C++ 和 Lua 之间的自动绑定功能。

### 2. 集成Lua库

首先需要将 Lua 解释器集成到引擎中。你可以通过以下方式之一来集成 Lua ：
- **静态链接**：将 Lua 的源代码直接编译到引擎的代码中。你需要将 Lua 源文件（如 `lua.c`, `lauxlib.c` ， `lualib.c`）添加到引擎的构建中。
- **动态链接**：将 Lua 编译成动态链接库（ `.dll`  /  `.so` ），然后在运行时加载 Lua 库。

### 3. 创建Lua状态机（`lua_State`）

Lua 的核心是 `lua_State`，它代表了一个 Lua 虚拟机的状态。在游戏引擎中，你需要创建并管理一个 `lua_State` 实例。
```cpp
#include "lua.hpp"

lua_State* L = luaL_newstate();  // 创建Lua状态机
luaL_openlibs(L);  // 打开Lua标准库
```

### 4. **绑定C++类到Lua**

通过绑定库（如 LuaBridge 、Sol2 等），你可以将 C++ 类和方法暴露给 Lua 脚本。例如，使用 Sol2 库进行绑定：
```cpp
#include <sol/sol.hpp>

class Player {
public:
    Player(std::string name) : name(name) {}
    void move(int x, int y) {
        std::cout << name << " moves to (" << x << ", " << y << ")" << std::endl;
    }
private:
    std::string name;
};

void bind(lua_State* L) {
    sol::state_view lua(L);  // 获取Lua状态机的视图
    lua.new_usertype<Player>("Player", 
        sol::constructors<Player(std::string)>(),
        "move", &Player::move
    );
}
```

### 5. 加载并执行 Lua 脚本

在游戏引擎中，你可以加载和执行Lua脚本。可以使用 `luaL_dofile` 来加载并执行一个Lua文件，或者使用 `luaL_dostring` 来执行字符串中的Lua代码。
```cpp
int result = luaL_dofile(L, "script.lua");  // 执行Lua文件
if (result != LUA_OK) {
    const char* error = lua_tostring(L, -1);
    std::cerr << "Error: " << error << std::endl;
}
```

### 6. 从 Lua 调用 C++ 方法

你可以通过 Lua 调用 C++ 方法。例如，创建一个 `Player` 对象并在 Lua 脚本中调用其方法：
```lua
player = Player("John")
player:move(10, 20)
```

### 7. 错误处理

在Lua中执行脚本时，需要进行错误处理。例如，当脚本出现错误时，你可以打印错误信息并进行相应的处理：
```cpp
if (lua_pcall(L, 0, LUA_MULTRET, 0) != LUA_OK) {
    const char* error = lua_tostring(L, -1);
    std::cerr << "Lua error: " << error << std::endl;
    lua_pop(L, 1);
}
```

### 8. 销毁 Lua 状态机

在引擎退出时，要记得销毁Lua状态机：
```cpp
lua_close(L);  // 销毁Lua状态机
```

### 9. 提供 Lua 绑定接口

根据游戏引擎的需求，可能需要暴露更多的 C++ 功能到 Lua 中。例如，提供游戏对象管理、时间控制、物理引擎等功能的 Lua 接口。