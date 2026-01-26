---
leetcode: LeetCode 994. Rotting Oranges
difficulties: MEDIUM
link: https://leetcode.cn/problems/rotting-oranges
tags:
  - LeetCode
  - Hot100
---

## 题目

在给定的 `m x n` 网格 `grid` 中，每个单元格可以有以下三个值之一：

- 值 `0` 代表空单元格；
- 值 `1` 代表新鲜橘子；
- 值 `2` 代表腐烂的橘子。

每分钟，腐烂的橘子 **周围 4 个方向上相邻** 的新鲜橘子都会腐烂。

返回直到单元格中没有新鲜橘子为止所必须经过的最小分钟数。如果不可能，返回 `-1` 。

## 题解

### C++

```cpp
class Solution {
public:
    int orangesRotting(vector<vector<int>>& grid) {
        queue<pair<int, int>> q;
        int dirs[4][2] = {{0, 1}, {0, -1}, {1, 0}, {-1, 0}};
        int m = grid.size(), n = grid[0].size();
        int cnt = 0;
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (grid[i][j] == 2)
                    q.push({i, j});
                else if (grid[i][j] == 1)
                    cnt++;
            }
        }
        if (q.empty() && cnt > 0)
            return -1;
        int ans = 0;
        while (!q.empty()) {
            int t = q.size();
            for (int k = 0; k < t ; ++k) {
                auto p = q.front();
                q.pop();
                for (auto dir : dirs) {
                    int x = p.first + dir[0];
                    int y = p.second + dir[1];
                    if (x >= 0 && x < m && y >= 0 && y < n && grid[x][y] == 1) {
                        grid[x][y] = 2;
                        q.push({x, y});
                        cnt--;
                    }
                }
            }
            if (!q.empty())
                ans ++;
        }
        if (cnt > 0)
            return -1;  
        return ans;
    }
};
```