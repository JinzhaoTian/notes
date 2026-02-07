---
leetcode: LeetCode 17. Letter Combinations of a Phone Number
difficulties: MEDIUM
link: https://leetcode.cn/problems/letter-combinations-of-a-phone-number
tags:
  - LeetCode
  - Hot100
  - 回溯
---

## 题目

给定一个仅包含数字 `2-9` 的字符串，返回所有它能表示的字母组合。答案可以按**任意顺序**返回。


## 题解

### C++

```cpp
class Solution {
public:
    unordered_map<char, string> phoneMap = {
        {'2', "abc"},
        {'3', "def"},
        {'4', "ghi"},
        {'5', "jkl"},
        {'6', "mno"},
        {'7', "pqrs"},
        {'8', "tuv"},
        {'9', "wxyz"}
    };

    vector<string> letterCombinations(string digits) {
        if (digits.empty()) {
            return {};
        }
        
        vector<string> ans;
        string combination;
        backtrack(ans, digits, 0, combination);
        return ans;
    }

    void backtrack(vector<string>& ans, const string& digits, int index, string& combination) {
        if (index == digits.length()) {
            ans.push_back(combination);
        } else {
            char digit = digits[index];
            const string& letters = phoneMap.at(digit);
            for (const char& letter: letters) {
                combination.push_back(letter);
                backtrack(ans, digits, index + 1, combination);
                combination.pop_back();
            }
        }
    }
};
```