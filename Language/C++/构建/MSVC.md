是微软 Windows 平台 Visual Studio 自带的 C/C++ 编译器。对 Windows 平台支持好，编译快，但是对 C++ 的新标准支持的少。


## 链接



### `*.ilk` 文件

`*.ilk` 文件是增量链接文件（Incremental Link File），文件名通常与你的项目输出（如 `.exe` 或 `.dll`）名称相同，是由 Visual Studio 中的链接器（Linker）生成（当启用了增量链接这个编译选项时）。

> [!info] 可以安全删除
> 删除 `*.ilk` 文件本身是安全的，当下次重新构建项目时，链接器会发现没有 `*.ilk` 文件，它会自动进行一次完整的链接，并重新生成一个新的 `*.ilk` 文件。
> 
> 当你要发布版本时，不需要附带这个文件。

#### 关闭生成

如果你不希望它占用空间，可以在项目设置中关闭增量链接选项，步骤如下（以 Visual Studio 2022 为例）：
1. 右键点击你的项目 -> **属性** (Properties)。
2. 在配置属性 (Configuration Properties) 下，选择链接器（Linker）
3. 选择常规（General）
4. 找到启用增量链接（Enable Incremental Linking）选项，将其设置为否 (`/INCREMENTAL:NO`)。

关闭后，编译器将不再生成 `*.ilk` 文件，但每次链接都会是完整链接，速度可能会变慢。
