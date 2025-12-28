
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

![](../_imgs/Pasted%20image%2020230703155825.png)

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

### vector
头文件：`<vector>`，定义：可变长的对象数组。


## ③ 算法库

[API Reference](https://www.apiref.com/cpp-zh/cpp/algorithm.html)
标准库提供超过100个算法，常用有：count()、find()、swap()、reverse()、sort()、lower_bound()、upper_bound()、max()、min()

#### lambda表达式


## ④ 动态内存

