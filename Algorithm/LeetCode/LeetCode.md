
# 基本

## 1. 数组

## 2. 字符串

# 算法

## 1. 动态规划

> 动态规划要求问题的子问题具有最优子结构，同时又保留了子问题的解，避免重复计算。

1. [LeetCode 1143. Longest Common Subsequence](题解/LeetCode%201143.%20Longest%20Common%20Subsequence.md)

谈起动态规划，就不得不提到最长公共子序列。

2. [LeetCode 44. Wildcard Matching](题解/LeetCode%2044.%20Wildcard%20Matching.md)


## 2. 二分查找


## 3. 排序

> 排序算法接触了很多，排序算法是最经典的算法知识。因为其实现代码短，应该广，在面试中经常会问到排序算法及其相关的问题。一般在面试中最常考的是快速排序和归并排序等基本的排序算法，复习一下，总共有：冒泡排序、选择排序、插入排序、**归并排序**、**快速排序**、**堆排序**、希尔排序、计数排序、**桶排序**、基数排序。

1. [LeetCode 215. Kth Largest Element in an Array](题解/LeetCode%20215.%20Kth%20Largest%20Element%20in%20an%20Array.md)

2. [LeetCode 347. Top K Frequent Elements](题解/LeetCode%20347.%20Top%20K%20Frequent%20Elements.md)

3. [LeetCode 451. Sort Characters By Frequency](题解/LeetCode%20451.%20Sort%20Characters%20By%20Frequency.md)

4. [LeetCode 75. Sort Colors](题解/LeetCode%2075.%20Sort%20Colors.md)


## 4. 贪心思想

> 贪心法的核心思想是每次都选取局部最优的结果，最终形成全局最优。这种思想简单，易于实现，但是并不是所有具有最优子结构的问题都能用贪心，还可能会是动态规划。

1. [LeetCode 455. Assign Cookies](题解/LeetCode%20455.%20Assign%20Cookies.md)


## 5. 搜索

1. [LeetCode 1654. Minimum Jumps to Reach Home](题解/LeetCode%201654.%20Minimum%20Jumps%20to%20Reach%20Home.md)


## 6. 快慢指针

无向图中找环的一个经典算法：快慢指针

1. [LeetCode 141. Linked List Cycle](题解/LeetCode%20141.%20Linked%20List%20Cycle.md)


# 基础数据结构


## 1. 链表

1. [LeetCode 206. Reverse Linked List](题解/LeetCode%20206.%20Reverse%20Linked%20List.md)
2. [剑指 Offer 35. 复杂链表的复制](https://leetcode-cn.com/problems/fu-za-lian-biao-de-fu-zhi-lcof/)

这个链表不仅包括一个next指针，还包括一个random指针随机指向链表中的一个节点。所以深拷贝的时候就不能顺序遍历，可以使用的方法是：

1. 方法一：先遍历一遍，借助hashmap，把新节点创造出来，然后再遍历一遍，把next指针和random指针连上。时间复杂度O(n)，空间O(n)。
2. 方法二：拼接+拆分。时间复杂度O(n)，空间O(1)。

## 2. 树

1. [LeetCode 449. Serialize and Deserialize BST](题解/LeetCode%20449.%20Serialize%20and%20Deserialize%20BST.md)

2. [LeetCode 102. Binary Tree Level Order Traversal](题解/LeetCode%20102.%20Binary%20Tree%20Level%20Order%20Traversal.md)

3. [LeetCode 96. Unique Binary Search Trees](题解/LeetCode%2096.%20Unique%20Binary%20Search%20Trees.md)


## 3. 栈

## 4. 堆（优先队列）

## 5. 图

## 6. 链表

## 7. 有序集合

## 8. 哈希表




# 高级数据结构

## 1. 并查集

## 2. 后缀数组

# 技巧

## 1. 位运算

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

## 2. 双指针

> 双指针主要用于遍历数组，两个指针指向不同的元素，从而协同完成任务。双指针法可以有助于减少遍历次数。

1. [LeetCode 167. Two Sum II - Input Array Is Sorted](题解/LeetCode%20167.%20Two%20Sum%20II%20-%20Input%20Array%20Is%20Sorted.md)

2. [LeetCode 633. Sum of Square Numbers](题解/LeetCode%20633.%20Sum%20of%20Square%20Numbers.md)

3. [LeetCode 345. Reverse Vowels of a String](题解/LeetCode%20345.%20Reverse%20Vowels%20of%20a%20String.md)

4. [LeetCode 680. Valid Palindrome II](题解/LeetCode%20680.%20Valid%20Palindrome%20II.md)

5. [LeetCode 88. Merge Sorted Array](题解/LeetCode%2088.%20Merge%20Sorted%20Array.md)

6. [LeetCode 141. Linked List Cycle](题解/LeetCode%20141.%20Linked%20List%20Cycle.md)

7. [LeetCode 524. Longest Word in Dictionary through Deleting](题解/LeetCode%20524.%20Longest%20Word%20in%20Dictionary%20through%20Deleting.md)




# 数学

## 1. 数论

## 2. 几何

# 其他

## 1. 多线程

## 2. 数据库


# Reference

1.  [GitHub CS Note](https://github.com/CyC2018/CS-Notes) 
2. [十大排序算法](https://zhuanlan.zhihu.com/p/42586566) 
