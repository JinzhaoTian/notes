[Leetcode 1654. Minimum Jumps to Reach Home](https://leetcode-cn.com/problems/minimum-jumps-to-reach-home/)

一个虫最少要经过多少跳才能到家。

DFS:

```
class Solution:
    def minimumJumps(self, forbidden: List[int], a: int, b: int, x: int) -> int:
        forbidden = set(forbidden)
        self.res = -1

        def dfs(pos, cnt, back):
            if self.res < 0 and 0 <= pos <= 6000:
                if pos == x:
                    self.res = cnt
                    return
                if pos + a not in forbidden:
                    forbidden.add(pos + a)  # 防止无限递归，比如 a = b 时，不加限制，就会出现无限递归
                    dfs(pos + a, cnt + 1, 0)
                if pos - b not in forbidden and back != 1: # 若back为1说明上次就是往后跳的
                    dfs(pos - b, cnt + 1, 1)
        
        dfs(0, 0, 0)
        return self.res
```

BFS:

```
class Solution:
    def minimumJumps(self, forbidden: List[int], a: int, b: int, x: int) -> int:
        forbidden = set(forbidden)
        que = deque()
        que.append((0, 0, False))

        while que:
            pos, cnt, back = que.popleft()
            if pos == x:
                return cnt
            if pos + a < 6000 and pos + a not in forbidden:
                forbidden.add(pos + a)
                que.append((pos + a, cnt + 1, False))
            if pos - b > 0 and not back and pos - b not in forbidden:
                que.append((pos - b, cnt + 1, True))
        return -1
```

