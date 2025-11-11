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

