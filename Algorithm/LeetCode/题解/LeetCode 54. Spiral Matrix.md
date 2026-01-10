---
leetcode: LeetCode 54. Spiral Matrix
difficulties: MEDIUM
link: https://leetcode.cn/problems/spiral-matrix
tags:
  - LeetCode
  - Hot100
---

## 题目

给你一个 `m` 行 `n` 列的矩阵 `matrix` ，请按照顺时针螺旋顺序，返回矩阵中的所有元素。

## 题解

### C++

```cpp
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        if (matrix.size() == 0 || matrix[0].size() == 0)
            return {};

        int m = matrix.size(), n = matrix[0].size();
        vector<vector<int>> directions = {{0, 1}, {1, 0}, {0, -1}, {-1, 0}};
        vector<vector<int>> visited(m, vector<int>(n));
        vector<int> ans(m * n);
        int row = 0, col = 0, dir = 0;
        for (int i = 0; i < m * n; i++) {
            ans[i] = matrix[row][col];
            visited[row][col] = 1;
            int nextRow = row + directions[dir][0], nextCol = col + directions[dir][1];
            if (nextRow < 0 || nextRow >= m || nextCol < 0 || nextCol >= n || visited[nextRow][nextCol] == 1) {
                dir = (dir + 1) % 4;
            }
            row += directions[dir][0];
            col += directions[dir][1];
        }
        return ans;
    }
};
```