
[API Reference](https://www.apiref.com/cpp-zh/cpp/string/basic_string.html)

STL 可以被划分为4个概念库：

1. **容器库(Contaienrs Library)** - 定义了管理和存储数据的容器。这个库的类模板在以下几个头文件中：array, string, vector, stack, queue, deque, list, forward_list, set, unordered_set, map以及unordered_map。
	1. 顺序容器：vector, string, deque, list, forward_list, array
	2. 关联容器
		1. 有序：map, set, multimap, multiset
		2. 无序：unordered_map, unordered_set, unordered_multimap, unordered_multiset
2. **迭代器库(Iterators Library)** - 定义了迭代器，迭代器是类似指针的对象，通常被用于引用容器类的对象序列，这个库的内容定义在iterator中。
3. **算法库(Algorithms Library)** - 定义了一些使用比较广泛的通用算法，定义在algorithm头文件中。
4. **数值库(Numerics Library)** - 定义了一些通用的数学函数和一些对容器元素进行数值处理的操作，也包含一些用于随机数生成的高级函数。定义在complex, cmath, valarray, numeric, random, ratio 以及cfenv这些头文件中。


## ① IO库   

### 读写流
头文件：`<iostream>`，

### 字符串流

### 文件流



## ② 容器库

![](Pasted%20image%2020230703155825.png)

1. **set** - 基于**红黑树实现**，红黑树具有自动排序的功能，因此map内部所有的数据，在任何时候，都是有序的。
2. **unordered_set** - 基于**哈希表实现**，数据插入和查找的时间复杂度很低，几乎是常数时间，而代价是消耗比较多的内存，无自动排序功能。底层实现上，使用一个下标范围比较大的数组来存储元素，形成很多的桶，利用hash函数对key进行映射到不同区域进行保存。
3. **map** - 基于**红黑树实现**，该结构具有自动排序的功能，因此map内部的所有元素都是有序的，红黑树的每一个节点都代表着map的一个元素，因此，对于map进行的查找，删除，添加等一系列的操作都相当于是对红黑树进行这样的操作，故红黑树的效率决定了map的效率。
4. **unordered_map** - 基于**哈希表实现**，因此其元素的排列顺序是杂乱的，无序的。其查找速度非常的快。


### 通用操作

1. 迭代器：
	1. `.begin()`和`.end()`：返回指向首元素或尾元素的迭代器，是可变迭代器或者是const迭代器取决于容器类型。
	2. `.cbegin()`和`.cend()`：返回指向首元素或尾元素的常迭代器（const）。
	3. `.rbegin()`和`.rend()`，及`.crbegin()`和`.crend()`：返回指向首元素或尾元素的逆向迭代器。
2. 容量：
	1. `.empty()`：若容器为空则为 true ，否则为 false ，相当于是否 begin() == end() 。
	2. `.size()`：返回容器中的元素数，无符号整数size_type。
	3. `.max_size()`：返回容器中可容纳的最大元素数目。
3. 操作：
	1. `.clear()`：从容器中移除所有元素，非法化所有指针、引用及迭代器。
	2. `.insert(size_type index, )`：从下标位置插入元素。
	3. `.erase(size_type index = 0, size_type count = npos)`：根据输入参数的不同，删除从index开始的若干个元素。
	4. `.emplace(init)`：使用init构造一个容器中的元素
	5. `.swap()`：交换元素位置。

### string
头文件：`<string>`，定义：可变长的字符序列。

1. 定义和初始化
   默认初始化：`string s1;`，创建副本：`string s2(s1);`和`string s2 = s1;`、`string s3("value")`和`string s3 = "value"`、`string s4(10, 'c')`和`string s4 = string(10, 'c')`。
   其中，有等号是**拷贝初始化**，无等号是**直接初始化**。
2. 操作
	1. 输入输出：
		1. 从输入输出流读写`os << s;`、` is >> s;`，读取一行赋值给s；
		2. `getline`读写：
		   共同点都在于可以接收除回车符之外的空白字符（空格，Tab），并将回车符摒弃，自动转为`\0`。
			1. `cin.getline(char *cha, int num, char delim);`：属于istream流，定义在头文件`<iostream>`，**只能向字符数组进行输入**，输入过程中达到num-1个数或者提前遇到delim字符，则输入结束。
			2. `getline(istream& is, string& str, char delim)`：属于string流，定义在头文件`<string>`，从输入流读取字符并将它们放进string。**只能向string类进行输入**，遇到delim字符输入结束。
	2. 拼接：确保每个`+`两侧的运算对象至少有一个是string。
		1. 正确形式：`string s1 = "Hello,", s2 = "World\n";string s3 = s1 + s2;`，`s1 += s2;`，`string s4 = s1 + ", ";`，`string s6 = s1 + ", " + "World";`。
		2. 错误形式：`string s5 = "Hello" + ", ";`，`string s7 = "Hello" + ", " + s2;`
	3. 对象操作：
		1. 元素访问：
			1. `.at(size_type pos)`：返回根据字符位置pos指定的字符的引用。`string s = 'abc'; s.at(2) = 'x';`，常数复杂度。
			2. `.front()`：返回 string 中首字符的引用。`string s("Exemplary"); char& f = s.front();f = 'e';`
			3. `.back()`：返回字符串中的尾字符。
		2. 迭代器：
			1. `.begin()`和`.end()`：返回指向首字符或尾字符的迭代器，是可变迭代器或者是const迭代器取决于string对象。
			2. `.cbegin()`和`.cend()`：返回指向首字符或尾字符的常迭代器（const）。
			3. `.rbegin()`和`.rend()`，及`.crbegin()`和`.crend()`：返回指向首字符或尾字符的逆向迭代器。
		3. 容量：
			1. `.empty()`：若 string 为空则为 true ，否则为 false ，相当于是否 begin() == end() 。
			2. `.size()`和`.length()`：返回 string 中的元素数，无符号整数size_type。
		4. 操作：
			1. `.clear()`：从 string 中移除所有字符，非法化所有指针、引用及迭代器。
			2. `.insert(size_type index, string const& str)`：插入字符或者字符串，从下标位置。
			3. `.erase(size_type index = 0, size_type count = npos)`：根据输入参数的不同，删除从index开始的若干个字符。
			4. `.push_back(CharT ch)`：后附给定字符 `ch` 到字符串尾。
			5. `.pop_back()`：从字符串移除末字符。
			6. `.append(size_type count, CharT ch)`：根据参数的不同，后附额外字符到字符串。`s.append(3, '*');`、`s.append(str);`、`s.append(str, 3, 3);`、`s.append(begin(carr) + 3, end(carr));`
			7. `.compare(const string& str)`：比较此 string 与 str 。
			8. `.replace(size_type pos, size_type count, const string& str)`：以新字符串 str 替换 `[pos, pos + count)` 所指示的 string 部分。
			9. `.substr(size_type pos = 0, size_type count = npos)`：返回子串 `[pos, pos+count)` 。若请求的子串越过 string 的结尾，或若 count == npos ，则返回的子串为 `[pos, size())` 。
			10. `.copy(CharT* dest, size_type count, size_type pos = 0)`：复制子串 `[pos, pos+count)` 到 `dest` 所指向的字符串。若请求的子串越过 string 结尾，或若 count == npos ，则复制的子串为 `[pos, size())` 。实例：`string foo("quuuux");char bar[7]{};foo.copy(bar, sizeof bar);`
			11. `.resize(size_type count)`：重设 string 大小以含 `count` 个字符。
			12. `.swap(string& other);`：交换 string 与 `other` 的内容，可能非法化所有迭代器和引用。
			13. `.starts_with(const string& str)`：检查 string 是否始于给定前缀。
			13. `.ends_with(const string& str)`：
		5. 查找：
			1. `.find(const string& str, size_type pos = 0)`：寻找首个等于给定字符序列 str 的子串。搜索始于 `pos` ，即找到的子串必须不始于 `pos` 之前的位置。
			2. `.rfind(const string& str, size_type pos = 0)`：寻找等于给定字符序列 str 的最后子串。搜索始于 `pos` ，即找到的子串必须不始于 `pos` 后的位置。若将 npos 或任何不小于 size()-1 的值作为 `pos` 传递，则在整个字符串中搜索。
			3. `.find_first_of(const string& str, size_type pos = 0)`：寻找等于给定字符序列 str 中字符之一的首个字符。搜索只考虑区间 `[pos, size())` 。若区间中不存在字符，则返回 npos 。
			4. `.find_last_of(const string& str, size_type pos = npos)`：寻找等于给定字符序列中字符之一的最后字符。
		6. 数值转换：
			1. `stoi(const string& str, size_t* pos = 0, int base = 10)`、`stol()`、`stoll()`：转译字符串 `str` 中的有符号整数值。
			2. `stof(const string& str, size_t* pos = 0)`、`stod()`、`stold()`：转译 `str` 中的浮点值。
			3. `to_string(num)`：返回一个包含转换后值的字符串。

分割字符串：
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

### vector
头文件：`<vector>`，定义：可变长的对象数组。


## ③ 算法库

[API Reference](https://www.apiref.com/cpp-zh/cpp/algorithm.html)
标准库提供超过100个算法，常用有：count()、find()、swap()、reverse()、sort()、lower_bound()、upper_bound()、max()、min()

#### lambda表达式


## ④ 动态内存

