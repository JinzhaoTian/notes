


## 常用类

### `std::filesystem::path`

`std::filesystem::path` 是一个专门用于表示和操作文件系统路径的类，是 C++ 17 文件系统库的核心，提供了跨平台、安全、便捷的路径处理能力。

#### 关键特性

1. **平台自适应**
	- `path` 对象在内部以一种与操作系统无关的方式存储路径。
	- 当需要输出或转换为字符串时，它会自动根据当前平台生成正确的格式（Windows 用 `\`，Linux/macOS 用 `/`）。
	- **重要**：在代码中，你可以**安全地使用正斜杠** `/` 作为字面量路径的分隔符，它在所有平台上都能被 `path` 正确解析。
```cpp
fs::path p1 = "C:/Users/Name/Document/file.txt"; // 在Windows上也能正确工作
fs::path p2 = "/home/user/document/file.txt";    // 在Linux/macOS上使用
```

2. **路径语法自动处理**
	- 它理解不同平台的根名称、根目录、相对路径和绝对路径的概念。
```cpp
fs::path p = "C:\\Windows\\System32\\drivers\\etc\\hosts";
// 在Windows上，p会被识别为绝对路径，根名为"C:"，根目录为"\\"
```

3. **丰富的成员函数**
	- `path` 提供了大量成员函数来分解和检查路径，而无需手动进行字符串操作。

#### 成员函数

1. **分解路径**：函数返回一个新的 `path` 对象，假设：`"C:\\Users\\Alice\\data\\config.json"`
	- `root_name()`：根名称，`"C:"`
	- `root_directory()`：根目录，`"\\"`
	- `root_path()`：根路径（以上两者结合），`"C:\\"`
	- `relative_path()`：根路径之后的部分，`"Users\\Alice\\data\\config.json"`
	- `parent_path()`：父路径，`"C:\\Users\\Alice\\data"`
	- `filename()`：文件名部分（含扩展名），`"config.json"`
	- `stem()`：主干名（不含扩展名)，`"config"`
	- `extension()`：扩展名（含点），`".json"`
	- **注意**：
		- 对于 `"/home/user/file.tar.gz"`，`stem()` 返回 `"file.tar"`，`extension()` 返回 `".gz"`。
		- 对于以点开头的文件（如 `".gitignore"`），`stem()` 返回空，`extension()` 返回 `".gitignore"`。

2. **检查路径元素**：返回 `bool`
	- `has_root_path()`：是否有根路径
	- `has_root_name()`：是否有根名称
	- `has_root_directory()`：是否有根目录
	- `has_relative_path()`：是否有相对路径部分
	- `has_parent_path()`：是否有父路径
	- `has_filename()`：是否有文件名
	- `has_stem()`：是否有主干名
	- `has_extension()`：是否有扩展名
	- `is_absolute()`：是否是绝对路径
	- `is_relative()`：是否是相对路径

3. **修改路径**
	- `clear()`：清空路径
	- `make_preferred()`：将目录分隔符转换为平台首选形式
	- `remove_filename()`：移除文件名部分
	- `replace_filename(const path& p)`：替换文件名部分
	- `replace_extension(const path& p = path())`：替换扩展名（为空则移除）
	- `operator/=`：追加路径（自动添加分隔符）

4. **转换和比较**
	- `string()`：转换为本地编码的 `std::string`
	- `wstring()`：转换为 `std::wstring`
	- `u8string()`：转换为 UTF-8 编码的 `std::string`（C++20 后返回 `std::u8string`）
	- `generic_string()`：转换为通用格式（使用 `/`）的字符串
	- `c_str()`：转换为 C 风格字符串
	- `operator==`， `operator!=`，`operator<`：比较路径


### `std::filesystem::directory_entry`

`std::filesystem::directory_entry` 是表示文件系统中的一个目录项（即一个文件、目录、符号链接等）的快照或缓存的一个类。

> [!tip] 设计哲学
> 在遍历目录时，一次性获取文件最基本、最常用的元数据并缓存起来，后续的查询可以直接使用缓存，避免重复的系统调用。

该对象存储一个 `path` 作为成员，并可能也在目录迭代过程中存储附带的文件属性（硬链接数、状态、符号链接状态、文件大小、及最后写入时间）。

#### 成员函数

1. **获取路径**
	- `const path& path() const noexcept`：返回该目录项对应的路径对象，这是它最基本的信息。

2. **查询文件状态和类型（通常使用缓存，效率高）**
	- `bool exists() const`：检查该目录项是否存在。
	- `bool is_regular_file() const`：检查是否是常规文件。
	- `bool is_directory() const`：检查是否是目录。
	- `bool is_symlink() const`：检查是否是符号链接。
	- `bool is_block_file()`
	- `bool is_character_file()`
	- `bool is_fifo()`
	- `bool is_socket()` ：检查其他特殊文件类型。
	- `file_status status() const`：返回文件的详细状态（包含类型和权限）。遵循符号链接。
	- `file_status symlink_status() const`：返回文件的状态，但对于符号链接，它返回的是符号链接本身的状态，而不是其指向的目标。

3. **获取文件属性（通常使用缓存，效率高）**
	- `uintmax_t file_size() const`：返回文件的大小（以字节为单位）。对于非常规文件（如目录）行为未定义。
	- `std::chrono::time_point<std::chrono::file_clock> last_write_time() const`：返回文件最后一次被修改的时间。

4. **刷新缓存**
	- `void refresh()`：如果自从创建这个 `directory_entry` 对象后，文件系统的状态可能已经改变了（例如，文件被删除或大小改变），你可以调用此函数来强制刷新缓存，使其与磁盘上的实际状态同步。

5. **比较操作**：
	- 支持 `==`，`!=`，`<` 等比较操作符，通常基于其封装的 `path` 进行比较。


### `std::filesystem::directory_iterator`

通常不会直接创建 `directory_entry` 对象，而是通过 `std::filesystem::directory_iterator` 获取文件系统目录中文件的迭代器容器，其元素为 `directory_entry` 对象（可用于遍历目录）。

```cpp
for (const auto& entry : std::filesystem::directory_iterator(".")) {
	// 这里的 `entry` 就是一个 directory_entry 对象
	std::cout << "File name: " << entry.path() << std::endl;
}
```

#### 问题

1. Windows 下使用带有正斜杠的文件路径，调用 `std::filesystem::directory_iterator()` 时会报错，而在 macOS 下无问题




## 跨平台实现差异

`std::filesystem::path` 在不同操作系统上有不同的内部实现和处理方式。

```cpp
#include <iostream>
#include <filesystem>
#include <iomanip>

namespace fs = std::filesystem;

void displayPathInfo(const fs::path& p, const std::string& title) {
    std::cout << "\n" << title << ":\n";
    std::cout << "  string(): " << p.string() << "\n";
    std::cout << "  generic_string(): " << p.generic_string() << "\n";
    
    #ifdef _WIN32
    std::cout << "  native(): " << p.native() << "\n"; // wstring on Windows
    #endif
    
    std::cout << "  root_name(): " << p.root_name() << "\n";
    std::cout << "  root_directory(): " << p.root_directory() << "\n";
    std::cout << "  root_path(): " << p.root_path() << "\n";
    std::cout << "  relative_path(): " << p.relative_path() << "\n";
    std::cout << "  parent_path(): " << p.parent_path() << "\n";
    std::cout << "  filename(): " << p.filename() << "\n";
    std::cout << "  extension(): " << p.extension() << "\n";
}

int main() {
    // 创建示例路径
    fs::path path1 = "/usr/include/stdc++.h";
    fs::path path2 = "C:\\Windows\\System32\\kernel32.dll";
    fs::path path3 = "相对路径/示例.txt";
    
    // 显示路径信息
    displayPathInfo(path1, "Unix风格路径");
    displayPathInfo(path2, "Windows风格路径");
    displayPathInfo(path3, "相对路径");
    
    // 演示路径转换
    std::cout << "\n=== 路径转换演示 ===\n";
    fs::path path;
    auto str = path.string();
    auto other = fs::path(str);
    
    std::cout << "空路径转换: " << other << "\n";
    
    // 不同平台的路径分隔符
    std::cout << "\n偏好路径分隔符: " << fs::path::preferred_separator << "\n";
    
    return 0;
}
```


### 差异点说明


#### Windows

1. 使用反斜杠 `\` 作为路径分隔符（但也支持正斜杠 `/`）
2. 支持驱动器号（如 `C:`）
3. 支持 UNC 路径（如 `\\server\share`）
4. 内部使用 `wchar_t` 存储路径，支持 Unicode


#### macOS/Linux（Unix-like）

1. 使用正斜杠 `/` 作为路径分隔符
2. 没有驱动器号概念，使用单一的根目录 `/`
3. 路径区分大小写
4. 内部使用 `char` 存储路径（UTF-8 编码）