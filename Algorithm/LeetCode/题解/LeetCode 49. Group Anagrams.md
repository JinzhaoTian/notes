---
leetcode: LeetCode 49. Group Anagrams
difficulties: MEDIUM
link: https://leetcode.cn/problems/group-anagrams
tags:
  - LeetCode
  - 字符串
  - Hot100
---

## 题目

给你一个字符串数组，请你将字母异位词组合在一起。


**示例 1:**

**输入:** `strs = ["eat", "tea", "tan", "ate", "nat", "bat"]`

**输出:** `[["bat"],["nat","tan"],["ate","eat","tea"]]`

**解释：**

- 在 strs 中没有字符串可以通过重新排列来形成 `"bat"`。
- 字符串 `"nat"` 和 `"tan"` 是字母异位词，因为它们可以重新排列以形成彼此。
- 字符串 `"ate"` ，`"eat"` 和 `"tea"` 是字母异位词，因为它们可以重新排列以形成彼此。


## 题解

### C++

```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> mp;
        for (string& str: strs){
            string key = str;
            sort(key.begin(), key.end());
            mp[key].emplace_back(str);
        }

        vector<vector<string>> ans;
        for (auto it = mp.begin(); it != mp.end(); ++it){
            ans.emplace_back(it -> second);
        }
        return ans;
    }
};
```
