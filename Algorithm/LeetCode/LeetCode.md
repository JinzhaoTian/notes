
## 数据结构

### 数组

### 字符串

1. [字符串基础原理](../Basic%20Algorithm/Data%20Structure/字符串.md)


### 链表

这个链表不仅包括一个next指针，还包括一个random指针随机指向链表中的一个节点。所以深拷贝的时候就不能顺序遍历，可以使用的方法是：

1. 方法一：先遍历一遍，借助hashmap，把新节点创造出来，然后再遍历一遍，把next指针和random指针连上。时间复杂度O(n)，空间O(n)。
2. 方法二：拼接+拆分。时间复杂度O(n)，空间O(1)。


```base
filters:
  and:
    - "!leetcode.isEmpty()"
    - file.tags.containsAny("链表", "双向链表")
properties:
  note.leetcode:
    displayName: 题目
  note.difficulties:
    displayName: 难度
  note.link:
    displayName: 链接
views:
  - type: table
    name: 链表
    order:
      - file.name
      - difficulties
      - link
    sort:
      - property: leetcode
        direction: ASC
    columnSize:
      file.name: 259
      note.difficulties: 84

```


### 树



```base
filters:
  and:
    - "!leetcode.isEmpty()"
    - file.tags.containsAny("二叉树", "树")
properties:
  note.leetcode:
    displayName: 题目
  note.difficulties:
    displayName: 难度
  note.link:
    displayName: 链接
views:
  - type: table
    name: 树
    order:
      - file.name
      - difficulties
      - link
    sort:
      - property: leetcode
        direction: ASC
    columnSize:
      file.name: 259
      note.difficulties: 84

```


### 栈

### 堆

### 优先队列


### 双端队列



### 哈希表

### 图

### 并查集

### 后缀数组


## 算法

### 动态规划

动态规划要求问题的子问题具有最优子结构，同时又保留了子问题的解，避免重复计算。


```base
filters:
  and:
    - "!leetcode.isEmpty()"
    - file.tags.containsAny("#动态规划")
properties:
  note.leetcode:
    displayName: 题目
  note.difficulties:
    displayName: 难度
  note.link:
    displayName: 链接
views:
  - type: table
    name: 动态规划
    order:
      - file.name
      - difficulties
      - link
    sort:
      - property: leetcode
        direction: ASC
    columnSize:
      file.name: 259
      note.difficulties: 84
```


### 二分查找


```base
filters:
  and:
    - "!leetcode.isEmpty()"
    - file.tags.containsAny("二分查找")
properties:
  note.leetcode:
    displayName: 题目
  note.difficulties:
    displayName: 难度
  note.link:
    displayName: 链接
views:
  - type: table
    name: 二分查找
    order:
      - file.name
      - difficulties
      - link
    sort:
      - property: leetcode
        direction: ASC
    columnSize:
      file.name: 259
      note.difficulties: 84

```


### 排序

排序算法接触了很多，排序算法是最经典的算法知识。因为其实现代码短，应该广，在面试中经常会问到排序算法及其相关的问题。一般在面试中最常考的是快速排序和归并排序等基本的排序算法，复习一下，总共有：冒泡排序、选择排序、插入排序、**归并排序**、**快速排序**、**堆排序**、希尔排序、计数排序、**桶排序**、基数排序。


```base
filters:
  and:
    - "!leetcode.isEmpty()"
    - file.tags.containsAny("排序")
properties:
  note.leetcode:
    displayName: 题目
  note.difficulties:
    displayName: 难度
  note.link:
    displayName: 链接
views:
  - type: table
    name: 排序
    order:
      - file.name
      - difficulties
      - link
    sort:
      - property: leetcode
        direction: ASC
    columnSize:
      file.name: 259
      note.difficulties: 84

```



### 贪心思想

贪心法的核心思想是每次都选取局部最优的结果，最终形成全局最优。这种思想简单，易于实现，但是并不是所有具有最优子结构的问题都能用贪心，还可能会是动态规划。


```base
filters:
  and:
    - "!leetcode.isEmpty()"
    - file.tags.containsAny("贪心")
properties:
  note.leetcode:
    displayName: 题目
  note.difficulties:
    displayName: 难度
  note.link:
    displayName: 链接
views:
  - type: table
    name: 贪心
    order:
      - file.name
      - difficulties
      - link
    sort:
      - property: leetcode
        direction: ASC
    columnSize:
      file.name: 259
      note.difficulties: 84

```




### 搜索


```base
filters:
  and:
    - "!leetcode.isEmpty()"
    - file.tags.containsAny("搜索")
properties:
  note.leetcode:
    displayName: 题目
  note.difficulties:
    displayName: 难度
  note.link:
    displayName: 链接
views:
  - type: table
    name: 搜索
    order:
      - file.name
      - difficulties
      - link
    sort:
      - property: leetcode
        direction: ASC
    columnSize:
      file.name: 259
      note.difficulties: 84

```




### 快慢指针

无向图中找环的一个经典算法：快慢指针



```base
filters:
  and:
    - "!leetcode.isEmpty()"
    - file.tags.containsAny("快慢指针")
properties:
  note.leetcode:
    displayName: 题目
  note.difficulties:
    displayName: 难度
  note.link:
    displayName: 链接
views:
  - type: table
    name: 快慢指针
    order:
      - file.name
      - difficulties
      - link
    sort:
      - property: leetcode
        direction: ASC
    columnSize:
      file.name: 259
      note.difficulties: 84

```



### 双指针

双指针主要用于遍历数组，两个指针指向不同的元素，从而协同完成任务。双指针法可以有助于减少遍历次数。


```base
filters:
  and:
    - "!leetcode.isEmpty()"
    - file.tags.containsAny("双指针")
properties:
  note.leetcode:
    displayName: 题目
  note.difficulties:
    displayName: 难度
  note.link:
    displayName: 链接
views:
  - type: table
    name: 双指针
    order:
      - file.name
      - difficulties
      - link
    sort:
      - property: leetcode
        direction: ASC
    columnSize:
      file.name: 259
      note.difficulties: 84

```

### 位运算

1. [剑指 Offer 15. 二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)

暴力找思路简单，但是位运算精妙无穷啊。

```c++
class Solution:
    def hammingWeight(self, n: int) -> int:
        ret = 0
        while n:
            n &= n - 1
            ret += 1
        return ret
```



## 数学

### 数论

### 几何

## 其他

### 多线程

### 数据库


# Reference

1.  [GitHub CS Note](https://github.com/CyC2018/CS-Notes) 
2. [十大排序算法](https://zhuanlan.zhihu.com/p/42586566) 
