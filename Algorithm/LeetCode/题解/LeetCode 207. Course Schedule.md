---
leetcode: LeetCode 207. Course Schedule
difficulties: MEDIUM
link: https://leetcode.cn/problems/course-schedule
tags:
  - LeetCode
  - Hot100
  - 拓扑排序
---

## 题目

你这个学期必须选修 `numCourses` 门课程，记为 `0` 到 `numCourses - 1` 。

在选修某些课程之前需要一些先修课程。 先修课程按数组 `prerequisites` 给出，其中 `prerequisites[i] = [ai, bi]` ，表示如果要学习课程 `ai` 则 **必须** 先学习课程  `bi` 。

- 例如，先修课程对 `[0, 1]` 表示：想要学习课程 `0` ，你需要先完成课程 `1` 。

请你判断是否可能完成所有课程的学习？如果可以，返回 `true` ；否则，返回 `false` 。

## 题解

### C++

```cpp
class Solution {
public:
    bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
        vector<int> inDegree(numCourses);
        unordered_map<int, vector<int>> map;
        for (int i = 0; i < prerequisites.size(); ++i) {
            inDegree[prerequisites[i][0]] ++;
            map[prerequisites[i][1]].push_back(prerequisites[i][0]);
        }

        queue<int> que;
        for (int i = 0; i < numCourses; ++i) {
            if (inDegree[i] == 0)
                que.push(i);
        }

        int cnt = 0;
        while (!que.empty()) {
            int selected = que.front();
            que.pop();
            cnt++;
            for (int i = 0; i < map[selected].size(); ++i) {
                if (inDegree[map[selected][i]] > 0) {
                    inDegree[map[selected][i]] --;
                    if (inDegree[map[selected][i]] == 0)
                        que.push(map[selected][i]);
                }
            }
        }
        return cnt == numCourses;       
    }
};
```


