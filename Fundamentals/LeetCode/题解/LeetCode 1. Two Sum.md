---
leetcode: LeetCode 1. Two Sum
difficulties: EASY
link: https://leetcode.cn/problems/two-sum
tags:
  - LeetCode
  - Hot100
---

## 分析

给出一个数组 `nums` ，给定一个目标值 `target` ，一眼看过去就有一个 $o(n^2)$ 的解法。

#### 普通解

两层 for 循环遍历，时间复杂度 $o(n^2)$ ，空间复杂度常量。

#### 高效解

首先想到排序 + 双指针，排序时间复杂度 $o(nlogn)$ ，如果双指针时间复杂度能在 $o(n)$ 那自然迎刃而解。但是在双指针的时候怎么滑动都不可能避免回退，所以此法不通。

主要想法是让遍历不进行回退，那么把遍历时的状态存下来，供后面的遍历查询使用，就可以降低时间复杂度。那么使用一个字典存下来是个自然的想法（因为有查询），所以使用哈希表。


## 复习

1. [排序算法](../../Basic%20Algorithm/算法与数据结构.md#排序)
2. 双指针
3. [哈希表](../../Basic%20Algorithm/算法与数据结构.md#哈希表)


## 题解

#### C++

1. 普通解：
```c++
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        for (int i = 0; i < nums.size() - 1; i++) {
            for (int j = i + 1; j < nums.size() ; j++){
                if (nums[i] + nums[j] == target){
                    return {i, j};
                }
            }
        }
        return {};
    }
};
```

2. 高效解：
```c++
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> hash_table;
        for (int i = 0; i < nums.size(); i++) {
            int left = target - nums[i];
            if (hash_table.find(left) != hash_table.end()){
                return {hash_table[left], i};
            }
            hash_table[nums[i]] = i;
        }
        return {};
    }
};
```

#### Java

1. 普通解
```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        for (int i = 0; i < nums.length; i++) {
            for (int j = i + 1; j < nums.length; j++) {
                if (nums[i] + nums[j] == target) {
                    return new int[] {i, j};
                }
            }
        }
        return new int[0];
    }
}
```

2. 高效解
```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> hash_table = new HashMap<Integer, Integer>();
        for (int i = 0; i < nums.length; ++i) {
            if (hash_table.containsKey(target - nums[i])) {
                return new int[]{ hash_table.get(target - nums[i]), i};
            }
            hash_table.put(nums[i], i);
        }
        return new int[0];
    }
}
```


#### JavaScript

#### TypeScript

#### Python

1. 普通解
```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        n = len(nums)
        for i in range(n):
            for j in range(i + 1, n):
                if nums[i] + nums[j] == target:
                    return [i, j]
        
        return []
```

2. 高效解
```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        hashtable = dict()
        for i, num in enumerate(nums):
            if target - num in hashtable:
                return [hashtable[target - num], i]
            hashtable[nums[i]] = i
        return []
```

#### C\#

1. 普通解
```c#

```