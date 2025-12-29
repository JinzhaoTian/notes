C++ 标准库的 `std::string` 是用来表示**字符串**的容器，头文件：`<string>`，但其实际上是**字节串**，并且不保留字符编码信息，以 `\0` 表示结束。

> [!tip] `std::string` 在某些场景下性能不够好
> 例如在涉及到系统 IO 的时候，通常会有如下接口：`std::size_t read(char* dst, std::size_t max_size);`，那么在为该接口分配缓冲区时，如果使用 `std::string`：
> ```cpp
> std::string buffer;
> buffer.resize(SIZE);
> auto read_size = read(buffer.data(), buffer.size());
> buffer.resize(read_size);
> ```
> 在某些场景（例如基础网络框架）下这么写会导致潜在的性能问题，可能会占用大量的CPU时间，因为 buffer 在 resize 的时候，不但**分配了内存还对这块内存做了初始化**，这导致了额外的 memset。


## 使用

### 定义和初始化

1. **默认初始化**：
```cpp
string s1;                      // s1 是一个空字符串
```

2. **创建副本**：
```cpp
string s2(s1);                  // s2 是 s1 的副本，直接初始化
string s2 = s1;                 // 等价于 s2(s1)，拷贝初始化

string s3("value");             // 直接初始化
string s3 = "value";            // 拷贝初始化

string s4(10, 'c');
string s4 = string(10, 'c');
```

其中，有等号是**拷贝初始化**，无等号是**直接初始化**。

### 操作

#### 输入输出

1. **标准控制台 I/O**：
	- `cin >> s`：自动跳过空格、换行符、制表符等，遇到下一个空白字符时停止，将读取的字符序列存入 `string` 对象，设置 `cin` 的状态标志（成功/失败）
	- `cout << s`：调用 `s.c_str()` 或 `s.data()` 获取字符数组，将字符逐个或批量写入**输出缓冲区**，遇到 `std::endl` 或缓冲区满时自动刷新到控制台。
```cpp
#include <iostream>
#include <string>

std::string s;

std::cin >> s;      // 从标准输入读取字符串
std::cout << s << std::endl;
```

2. **通用流 I/O**：`istream` 是所有输入流的基类，`ostream` 是所有输出流的基类，**无论流类型如何，读取和输出的逻辑相同**。
	- **常见流 I/O 类型**：
		- `iostream`：控制台输入输出流
		- `sstream`：字符串流
		- `fstream`：文件输入输出流
	- **`is >> s`**：
		- `is`是`istream&`引用
		- 重载 `operator>>(istream&, string&)` 模板函数
		- 流状态可检查（`is.good()`，`is.eof()` 等）
	- **`os << s`**：
		- `os`是`ostream&`引用
		- 重载 `operator<<(ostream&, const string&)` 模板函数
		- **不同流类型有不同的缓冲区策略**
```cpp
#include <iostream>
#include <sstream>
#include <fstream>

#include <string>

string s;

os << s;       // 将 s 写到输出流 os 中，返回 os
is >> s;       // 从 is 中读取字符串赋给 s，返回 is，以空白字符作为分隔
```

3. **`getline`**：可以接收除回车符之外的空白字符（空格，Tab），并将回车符摒弃，自动转为 `\0` 。
	- `cin.getline(char *cha, int num, char delim);`：属于 `istream` 流，定义在头文件`<iostream>`，**只能向字符数组进行输入**，输入过程中达到 `num-1` 个数或者提前遇到 `delim` 字符，则输入结束。
	- `getline(istream& is, string& str, char delim)`：属于 `string` 流，定义在头文件`<string>`，从输入流读取字符并将它们放进 `string`。**只能向 `string` 类进行输入**，遇到 `delim` 字符输入结束。

#### 比较


#### 操作

1. **拼接**：确保每个 `+` 两侧的运算对象至少有一个是 `string`。
```cpp
string s1 = "Hello,", s2 = "World\n";

string s3 = s1 + s2;
s1 += s2;
string s4 = s1 + ", ";
string s6 = s1 + ", " + "World";
```

2. **元素访问**：
	- `.at(size_type pos)`：返回根据字符位置pos指定的字符的引用。`string s = 'abc'; s.at(2) = 'x';`，常数复杂度。
	- `.front()`：返回 `string` 中首字符的引用。`string s("Exemplary"); char& f = s.front();f = 'e';`
	- `.back()`：返回字符串中的尾字符。
3. **迭代器**：
	- `.begin()`和`.end()`：返回指向首字符或尾字符的迭代器，是可变迭代器或者是const迭代器取决于string对象。
	- `.cbegin()`和`.cend()`：返回指向首字符或尾字符的常迭代器（const）。
	- `.rbegin()`和`.rend()`，及`.crbegin()`和`.crend()`：返回指向首字符或尾字符的逆向迭代器。
4. **容量**：
	- `.empty()`：若 string 为空则为 true ，否则为 false ，相当于是否 begin() == end() 。
	- `.size()`和`.length()`：返回 string 中的元素数，无符号整数size_type。
5. **操作**：
	- `.clear()`：从 string 中移除所有字符，非法化所有指针、引用及迭代器。
	- `.insert(size_type index, string const& str)`：插入字符或者字符串，从下标位置。
	- `.erase(size_type index = 0, size_type count = npos)`：根据输入参数的不同，删除从index开始的若干个字符。
	- `.push_back(CharT ch)`：后附给定字符 `ch` 到字符串尾。
	- `.pop_back()`：从字符串移除末字符。
	- `.append(size_type count, CharT ch)`：根据参数的不同，后附额外字符到字符串。`s.append(3, '*');`、`s.append(str);`、`s.append(str, 3, 3);`、`s.append(begin(carr) + 3, end(carr));`
	- `.compare(const string& str)`：比较此 string 与 str 。
	- `.replace(size_type pos, size_type count, const string& str)`：以新字符串 str 替换 `[pos, pos + count)` 所指示的 string 部分。
	- `.substr(size_type pos = 0, size_type count = npos)`：返回子串 `[pos, pos+count)` 。若请求的子串越过 string 的结尾，或若 count == npos ，则返回的子串为 `[pos, size())` 。
	- `.copy(CharT* dest, size_type count, size_type pos = 0)`：复制子串 `[pos, pos+count)` 到 `dest` 所指向的字符串。若请求的子串越过 string 结尾，或若 count == npos ，则复制的子串为 `[pos, size())` 。实例：`string foo("quuuux");char bar[7]{};foo.copy(bar, sizeof bar);`
	- `.resize(size_type count)`：重设 string 大小以含 `count` 个字符。
	- `.swap(string& other);`：交换 string 与 `other` 的内容，可能非法化所有迭代器和引用。
	- `.starts_with(const string& str)`：检查 string 是否始于给定前缀。
	- `.ends_with(const string& str)`：
6. **查找**：
	- `.find(const string& str, size_type pos = 0)`：寻找首个等于给定字符序列 str 的子串。搜索始于 `pos` ，即找到的子串必须不始于 `pos` 之前的位置。
	- `.rfind(const string& str, size_type pos = 0)`：寻找等于给定字符序列 str 的最后子串。搜索始于 `pos` ，即找到的子串必须不始于 `pos` 后的位置。若将 npos 或任何不小于 size()-1 的值作为 `pos` 传递，则在整个字符串中搜索。
	- `.find_first_of(const string& str, size_type pos = 0)`：寻找等于给定字符序列 str 中字符之一的首个字符。搜索只考虑区间 `[pos, size())` 。若区间中不存在字符，则返回 npos 。
	- `.find_last_of(const string& str, size_type pos = npos)`：寻找等于给定字符序列中字符之一的最后字符。
7. **数值转换**：
	- `stoi(const string& str, size_t* pos = 0, int base = 10)`、`stol()`、`stoll()`：转译字符串 `str` 中的有符号整数值。
	- `stof(const string& str, size_t* pos = 0)`、`stod()`、`stold()`：转译 `str` 中的浮点值。
	- `to_string(num)`：返回一个包含转换后值的字符串。
8. **分割字符串**：
```c++
vector<string> split(const string &str, const string &pattern)
{
    vector<string> res;
    if(str == "")
        return res;
    //在字符串末尾也加入分隔符，方便截取最后一段
    string strs = str + pattern;
    size_t pos = strs.find(pattern);

    while(pos != strs.npos)
    {
        string temp = strs.substr(0, pos);
        res.push_back(temp);
        //去掉已分割的字符串,在剩下的字符串中进行分割
        strs = strs.substr(pos+1, strs.size());
        pos = strs.find(pattern);
    }

    return res;
}
```

