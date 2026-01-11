---
leetcode: LeetCode 48. Rotate Image
difficulties: MEDIUM
link: https://leetcode.cn/problems/rotate-image
tags:
  - LeetCode
  - Hot100
---

## 题目

给定一个 `n × n` 的二维矩阵 `matrix` 表示一个图像，请你将图像顺时针旋转 90 度。

## 题解

### C++

```cpp
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();
        for (int i = 0; i < n / 2; ++i) {
            for (int j = 0; j < (n + 1) / 2; ++j) {
                int t = matrix[i][j];
                matrix[i][j] = matrix[n - j - 1][i];
                matrix[n - j - 1][i] = matrix[n - i - 1][n - j - 1];
                matrix[n - i - 1][n - j - 1] = matrix[j][n - i - 1];
                matrix[j][n - i - 1] = t;
            }
        }
    }
};
``` 