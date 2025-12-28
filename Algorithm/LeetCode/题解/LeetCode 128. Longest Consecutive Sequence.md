---
leetcode: LeetCode 128. Longest Consecutive Sequence
difficulties: MEDIUM
link: https://leetcode.cn/problems/longest-consecutive-sequence
tags:
  - LeetCode
  - Hot100
---

## 题目

给定一个未排序的整数数组 `nums` ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。

## 题解

### C++

```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        unordered_set<int> num_set;
        for (const int& num: nums) {
            num_set.insert(num);
        }

        int ans = 0;
        for (const int &num: num_set) {
            if (!num_set.count(num - 1)) {
                int currentNum = num;
                int currentLen = 1;
                while (num_set.count(currentNum + 1)) {
                    currentNum += 1;
                    currentLen += 1;
                }

                ans = max(ans, currentLen);
            }
        }

        return ans;
    }
};
```

