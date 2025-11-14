Unreal Engine 通过模块化的代码组织和清晰的目录隔离来区分 Editor（编辑器）和 Runtime（运行时）代码。

## 源码组织结构

1. **模块定义**：
	- `Source/[ModuleName]/Public/`：存放模块的公共头文件（`.h`），供其他模块引用。
	- `Source/[ModuleName]/Private/`：存放模块的实现文件（`.cpp`）和私有头文件。
	- `Source/[ModuleName]/[ModuleName].Build.cs`：定义模块的编译规则和依赖关系。

2. **运行时模块**：
	- `Source/[ModuleName]/[ModuleName]Module.cpp`：实现运行时模块的接口
	- `.uproject`：项目描述文件中的模块配置，控制运行时模块的加载时机
		- 如 `"LoadingPhase": "Default"`。

3. **编辑器模块**：
	- `Source/[ModuleName]Editor/`：编辑器专用模块的根目录，通常依赖对应的运行时模块。
	- `.uproject`：项目描述文件中的模块类型设置，将模块标记为编辑器类型
		- 如 `"Type": "Editor"`。


## 项目工程组织结构

```
MyGame/
├── Binaries/                         # 编译生成的二进制文件
├── Config/                           # 配置文件
│   ├── DefaultEditor.ini
│   ├── DefaultEngine.ini
│   └── DefaultGame.ini
├── Content/                          # 资源和蓝图文件
│   └── CustomPackage.uasset
├── Intermediate/                     # 中间缓存文件
│   ├── Build/
│   └── ...
├── Saved/                            # 自动保存文件
├── Source/                           # 游戏 C++ 源代码
│   ├── MyGame/
│   │   ├── MyGame.Build.cs
│   │   └── ...
│   ├── MyGame.Target.cs
│   └── MyGameEditor.Target.cs
└── MyGame.uproject
```


## 代码规范

Unreal Engine C++ 代码规范：

1. 命名清晰、明确、避免过度缩写
2. 变量名：大驼峰式（CamelCase），bool 类型须加 b 前缀
3. 类型名：前缀 + 大驼峰式
4. 引用传入可能修改的函数变量：加 Out 前缀
5. **开头字母**
	- `T`：模板类，如 `TMap`
	- `U`：继承自`U0bject`，如 `UMoviePlayerSettings`
	- `A`：继承自 `AActor`，如 `APlayerCameraManager`
	- `S`：继承自 `Swidget`，如 `SCompoundwidget`
	- `I`：抽象接口类，如 `INavNodeInterface`
	- `E`：枚举，如 `EAccountType`
	- `b`：布尔变量，如 `bHasFadedIn`
	- `F`：其他类，如 `FVector`

