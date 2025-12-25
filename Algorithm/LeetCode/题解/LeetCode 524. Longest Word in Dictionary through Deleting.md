---
leetcode: LeetCode 524. Longest Word in Dictionary through Deleting
difficulties: MEDIUM
link: https://leetcode-cn.com/problems/longest-word-in-dictionary-through-deleting
tags:
  - LeetCode
  - 字符串
  - 双指针
  - 排序
---

## 题目

给你一个字符串 `s` 和一个字符串数组 `dictionary` ，找出并返回 `dictionary` 中最长的字符串，该字符串可以通过删除 `s` 中的某些字符得到。

如果答案不止一个，返回长度最长且字母序最小的字符串。如果答案不存在，则返回空字符串。



## 题解

### C++

```cpp
class Solution {
public:
    string findLongestWord(string s, vector<string>& dictionary) {
        string res;
        for (int i = 0; i < dictionary.size(); i++){
            if (isSubstr(s, dictionary[i])){
                res = (res.length() < dictionary[i].length() || (res.length() == dictionary[i].length() && res > dictionary[i])) ? dictionary[i] : res;
            }
        }
        return res;
    }
    bool isSubstr(string s, string target){
        int i = 0, j = 0;
        while (i < s.length() && j < target.length()) {
            if (s[i] == target[j]) j ++;
            i ++;
        }
        return j == target.length();
    }
};
```

### Python

```python
class Solution:
    def findLongestWord(self, s: str, dictionary: List[str]) -> str:
        def isSubStr(s, target):
            i, j = 0, 0
            while i < len(s) and j < len(target):
                if (s[i] == target[j]):
                    j += 1
                i += 1
            return j == len(target)
        
        maxs = ''
        for target in dictionary:
            if isSubStr(s, target):
                maxs = target if len(maxs) < len(target) or (len(maxs) == len(target) and maxs > target) else maxs
        return maxs
```

