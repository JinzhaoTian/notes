---
leetcode: LeetCode 215. Kth Largest Element in an Array
difficulties: MEDIUM
link: https://leetcode-cn.com/problems/kth-largest-element-in-an-array
tags:
  - LeetCode
  - Hot100
  - 二分查找
  - 排序
---

## 题目

给定整数数组 `nums` 和整数 `k`，请返回数组中第 `k` 个最大的元素。

你必须设计并实现时间复杂度为 `O(n)` 的算法解决此问题。

## 分析

寻找第k个最大的元素，这个题目早就听说过了，但是一直对于解法怎么写不是很清楚。

**思路一**：最简单的想法就是，先对所有的元素进行排序，然后返回倒数第k个元素就好，如果使用的排序算法性能比较好，时间复杂度就可以达到O(nlogn)。

但是深入思考一下，还有可以做到时间复杂度更低的算法。

**思路二**：基于快速排序的思想。快排的每次partition都会返回pivot的确定位置，查看这个位置是不是所要求的位置。


**思路三**：基于堆排序。



## 题解

### C++

1. 快速排序：（正常写会超时）
```cpp
class Solution {
public:
    int findKthLargest(vector<int>& nums, int k) {
        int n = nums.size();
        return quickSelect(nums, 0, n - 1, n - k);
    }

    int quickSelect(vector<int>& nums, int l, int r, int k) {
        if (l == r)
            return nums[k];
        int partition = nums[l], i = l, j = r;
        while (i < j){
            while (i < j && nums[j] >= partition)
                j--;
            nums[i] = nums[j];
            while (i < j && nums[i] <= partition)
                i++;
            nums[j] = nums[i];
        }
        nums[i] = partition;
        if (k <= j)
            return quickSelect(nums, l, j, k);
        else
            return quickSelect(nums, j + 1, r, k);
    }
};
```

官方优化版：
```cpp
class Solution {
public:
    int findKthLargest(vector<int>& nums, int k) {
        int n = nums.size();
        return quickSelect(nums, 0, n - 1, n - k);
    }

    int quickSelect(vector<int>& nums, int l, int r, int k) {
        if (l == r)
            return nums[k];
        int partition = nums[l], i = l - 1, j = r + 1;
        while (i < j){
            do i++; while (nums[i] < partition);
            do j--; while (nums[j] > partition);
            if (i < j)
                swap(nums[i], nums[j]);
        }
        if (k <= j)
            return quickSelect(nums, l, j, k);
        else
            return quickSelect(nums, j + 1, r, k);
    }
};
```

### Python

1. 排序
```python
class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        nums.sort()
        return nums[len(nums) - k]
```

2. 快速排序
```python
class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        def partition(nums, left, right):
            pivot = nums[left]
            i, j =  left, right
            while i < j:
                while i < j and nums[j] >= pivot:
                    j -= 1
                nums[i] = nums[j]
                while i < j and nums[i] <= pivot:
                    i += 1
                nums[j] = nums[i]
            nums[i] = pivot
            return i
        
        def findKth(nums, ii, left, right):
            if left < right:
                index = partition(nums, left, right)
                if index == ii:
                    return
                elif index > ii:
                    findKth(nums, ii, left, index - 1)
                else:
                    findKth(nums, ii, index + 1, right)

        findKth(nums, len(nums) - k, 0, len(nums) - 1)
        return nums[len(nums) - k]
 
```

3. 堆排序：调用Python的API，很方便，但是总感觉有点...
```python
class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        n = len(nums)
        heap = nums[:]
        heapq.heapify(heap)  # 将heap原地变成小顶堆
        for _ in range(n - k):
            heapq.heappop(heap)
        return heap[0]
```
