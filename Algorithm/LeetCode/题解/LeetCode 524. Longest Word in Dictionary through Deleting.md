[524. Longest Word in Dictionary through Deleting](https://leetcode-cn.com/problems/longest-word-in-dictionary-through-deleting/)

Python:

```
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

C++:

```
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