---
leetcode: LeetCode 200. Number of Islands
difficulties: MEDIUM
link: https://leetcode.cn/problems/number-of-islands
tags:
  - LeetCode
  - Hot100
---

## 题目

给你一个由 `'1'`（陆地）和 `'0'`（水）组成的的二维网格，请你计算网格中岛屿的数量。

岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。

此外，你可以假设该网格的四条边均被水包围。

## 题解

### C++

```cpp
class Solution {
public:
    int numIslands(vector<vector<char>>& grid) {
        int m = grid.size();
        if (m == 0)
            return 0;
        int n = grid[0].size();

        int ans = 0;
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (grid[i][j] == '1') {
                    ans ++;
                    dfs(grid, i, j);
                }
            }
        }
        return ans;
    }

    void dfs(vector<vector<char>>& grid, int i, int j) {
        int m = grid.size(), n =grid[0].size();

        grid[i][j] = 0;
        if (i - 1 >= 0 && grid[i - 1][j] == '1') dfs(grid, i - 1, j);
        if (i + 1 < m  && grid[i + 1][j] == '1') dfs(grid, i + 1, j);
        if (j - 1 >= 0 && grid[i][j - 1] == '1') dfs(grid, i, j - 1);
        if (j + 1 < n  && grid[i][j + 1] == '1') dfs(grid, i, j + 1);
    }
};
```