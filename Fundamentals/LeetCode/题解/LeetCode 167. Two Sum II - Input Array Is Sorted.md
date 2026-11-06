---
leetcode: LeetCode 167. Two Sum II - Input Array Is Sorted
difficulties: MEDIUM
link: https://leetcode-cn.com/problems/two-sum-ii-input-array-is-sorted
tags:
  - LeetCode
  - 双指针
---

## 题目

给你一个下标从 **1** 开始的整数数组 `numbers` ，该数组已按 **非递减顺序排列**  ，请你从数组中找出满足相加之和等于目标数 `target` 的两个数。如果设这两个数分别是 `numbers[index1]` 和 `numbers[index2]` ，则 `1 <= index1 < index2 <= numbers.length` 。

以长度为 2 的整数数组 `[index1, index2]` 的形式返回这两个整数的下标 `index1` 和 `index2`。


## 题解

### C++

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& numbers, int target) {
        int l = 0, r =numbers.size() - 1;
        while(l < r){
            if(numbers[l] + numbers[r] > target){
                r--;
            } else if(numbers[l] + numbers[r] < target) {
                l++;
            } else {
                break;
            }
        }
        return {l + 1, r + 1};
    }
};
```


### Python

```python
class Solution:
    def twoSum(self, numbers: List[int], target: int) -> List[int]:
        l, r = 0, len(numbers)-1
        while l<r:
            if numbers[l] + numbers[r] > target:
                r -= 1
            elif numbers[l] + numbers[r] < target:
                l += 1
            else:
                break
        return [l+1, r+1]
```

