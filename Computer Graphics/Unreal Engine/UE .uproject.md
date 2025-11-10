`.uproject` 是 Unreal Engine 专用的项目文件格式，它是虚幻引擎项目的核心配置文件。

## 文件概述

`.uproject` 是一个 JSON 格式的文本文件，作为虚幻引擎项目的入口点和配置中心。

```json
{
	"FileVersion": 3,
	"EngineAssociation": "5.3",
	"Category": "Games",
	"Description": "A third-person action adventure game",
	"Modules": [
		{
			"Name": "AdventureGame",
			"Type": "Runtime",
			"LoadingPhase": "Default",
			"AdditionalDependencies": [
				"Engine",
				"CoreUObject",
				"InputCore",
				"UMG"
			]
		},
		{
			"Name": "AdventureGameEditor",
			"Type": "Editor",
			"LoadingPhase": "PostEngineInit"
		}
	],
	"Plugins": [
		{
			"Name": "Wwise",
			"Enabled": true
		},
		{
			"Name": "NvidiaDLSS",
			"Enabled": true
		}
	],
	"TargetPlatforms": ["Win64", "XSX", "PS5"],
	"EpicApp": "AdventureGame",
	"PreBuildSteps": {
		"Win64": "echo \"Running pre-build steps\""
	},
	"PostBuildSteps": {
		"Win64": "echo \"Running post-build steps\""
	}
}
```


## 项目组织

Unreal Engine 项目目录结构如下：

```cpp
AdventureGame/
├── Content/                        # 资源文件（蓝图、材质等）
├── Source/                         # C++ 源代码
│   ├── AdventureGame/              # 主游戏模块
│   │   ├── AdventureGame.Build.cs
│   │   ├── AdventureGame.h
│   │   └── AdventureGame.cpp
│   └── AdventureGameEditor/        # 编辑器模块
├── Config/                         # 配置文件
│   ├── DefaultEngine.ini
│   ├── DefaultGame.ini
│   └── DefaultEditor.ini
├── Binaries/                       # 编译输出
└── AdventureGame.uproject          # 项目配置文件
```

## 开发工作流

1. **项目识别和启动**
```bash
# 双击 .uproject 文件启动编辑器
AdventureGame.uproject

# 或通过命令行
UnrealEditor.exe AdventureGame.uproject
```

2. **生成项目文件**
```bash
# 生成 Visual Studio 解决方案
UnrealBuildTool -projectfiles -project="AdventureGame.uproject" -game -rocket -progress
```

3. **构建和打包**
```bash
# 构建项目
UnrealBuildTool AdventureGame Win64 Development -Project="AdventureGame.uproject"

# 打包项目
UnrealEditor.exe AdventureGame.uproject -run=Cook -TargetPlatform=Win64 -fileopenlog -unversioned -iterative
```

