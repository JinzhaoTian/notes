---
leetcode: LeetCode 438. Find All Anagrams in a String
difficulties: MEDIUM
link: https://leetcode.cn/problems/find-all-anagrams-in-a-string
tags:
  - LeetCode
  - Hot100
  - 字符串
  - 滑动窗口
---

## 题目

给定两个字符串 `s` 和 `p`，找到 `s` 中所有 `p` 的异位词的子串，返回这些子串的起始索引。

## 题解

### C++

```cpp
class Solution {
public:
    vector<int> findAnagrams(string s, string p) {
        int sn = s.size(), pn = p.size();

        if (sn < pn) {
            return vector<int>();
        }

        vector<int> ans;
        vector<int> sCnt(26), pCnt(26);
        for (int i = 0; i < pn; ++i) {
            sCnt[s[i] - 'a'] ++;
            pCnt[p[i] - 'a'] ++;
        }

        if (sCnt == pCnt) {
            ans.push_back(0);
        }

        for (int i = 0; i < sn - pn; ++i) {
            sCnt[s[i] - 'a']--;
            sCnt[s[i + pn] - 'a']++;

            if (sCnt == pCnt) {
                ans.push_back(i + 1);
            }
        }

        return ans;
    }
};
```