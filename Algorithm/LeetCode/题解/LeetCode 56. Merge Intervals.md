---
leetcode: LeetCode 56. Merge Intervals
difficulties: MEDIUM
link: https://leetcode.cn/problems/merge-intervals
tags:
  - LeetCode
  - Hot100
  - 排序
---

## 题目

以数组 `intervals` 表示若干个区间的集合，其中单个区间为 `intervals[i] = [starti, endi]` 。请你合并所有重叠的区间，并返回一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间。

## 题解

### C++

```cpp
class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        if (intervals.size() == 0) {
            return {};
        }
        sort(intervals.begin(), intervals.end());
        vector<vector<int>> ans;
        for (int i = 0; i < intervals.size(); ++i) {
            int l = intervals[i][0], r = intervals[i][1];
            if (!ans.size() || ans.back()[1] < l) {
                ans.push_back({l, r});
            } else {
                ans.back()[1] = max(ans.back()[1], r);
            }
        }
        return ans;
    }
};
```