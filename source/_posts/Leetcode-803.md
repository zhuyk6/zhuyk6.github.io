---
title: Leetcode 803
categories:
  - 刷题
  - Leetcode
tags: [并查集]
date: 2021-09-15 05:54:27
mathjax: false
---

[Leetcode 803](https://leetcode-cn.com/problems/bricks-falling-when-hit/)

<!--more-->

# Description

有一个 `m x n` 的二元网格，其中 `1` 表示砖块，`0` 表示空白。砖块 **稳定**（不会掉落）的前提是：

- 一块砖直接连接到网格的顶部，或者
- 至少有一块相邻（4 个方向之一）砖块 **稳定** 不会掉落时

给你一个数组 `hits` ，这是需要依次消除砖块的位置。每当消除 `hits[i] = (rowi, coli)` 位置上的砖块时，对应位置的砖块（若存在）会消失，然后其他的砖块可能因为这一消除操作而掉落。一旦砖块掉落，它会立即从网格中消失（即，它不会落在其他稳定的砖块上）。

返回一个数组 `result` ，其中 `result[i]` 表示第 `i` 次消除操作对应掉落的砖块数目。

**注意**，消除可能指向是没有砖块的空白位置，如果发生这种情况，则没有砖块掉落。

# Sample

**示例 1：**

```
输入：grid = [[1,0,0,0],[1,1,1,0]], hits = [[1,0]]
输出：[2]
解释：
网格开始为：
[[1,0,0,0]，
 [1,1,1,0]]
消除 (1,0) 处加粗的砖块，得到网格：
[[1,0,0,0]
 [0,1,1,0]]
两个加粗的砖不再稳定，因为它们不再与顶部相连，也不再与另一个稳定的砖相邻，因此它们将掉落。得到网格：
[[1,0,0,0],
 [0,0,0,0]]
因此，结果为 [2] 。
```

**示例 2：**

```
输入：grid = [[1,0,0,0],[1,1,0,0]], hits = [[1,1],[1,0]]
输出：[0,0]
解释：
网格开始为：
[[1,0,0,0],
 [1,1,0,0]]
消除 (1,1) 处加粗的砖块，得到网格：
[[1,0,0,0],
 [1,0,0,0]]
剩下的砖都很稳定，所以不会掉落。网格保持不变：
[[1,0,0,0], 
 [1,0,0,0]]
接下来消除 (1,0) 处加粗的砖块，得到网格：
[[1,0,0,0],
 [0,0,0,0]]
剩下的砖块仍然是稳定的，所以不会有砖块掉落。
因此，结果为 [0,0] 。
```

 

# Hint

- `m == grid.length`
- `n == grid[i].length`
- `1 <= m, n <= 200`
- `grid[i][j]` 为 `0` 或 `1`
- `1 <= hits.length <= 4 * 10^4`
- `hits[i].length == 2`
- `0 <= xi <= m - 1`
- `0 <= yi <= n - 1`
- 所有 `(xi, yi)` 互不相同

# Solution

很巧妙的题目，正常来说，需要维护一个数据结构，来不断的判断 “删除”结点之后图的连通性。

可复杂就复杂在，这是一张图，不是一颗树，删除某个结点后，可能仍然保持连通。

本题的精彩之处就在于 **逆序** 处理：倒过来添加结点。添加结点后连通块增大的部分，就是删除该结点后掉落的部分。

显然 “添加” 要远比 “删除” 简单的多，本题仅需要考虑连通性和连通块的大小，并查集就可以满足需求。

# Code

```python
class Solution:
    def hitBricks(self, grid: List[List[int]], hits: List[List[int]]) -> List[int]:
        m, n = len(grid), len(grid[0])

        fa = [i for i in range(m * n + 1)]
        size = [1 for i in range(m * n + 1)]

        def getfa(x: int) -> int:
            if x != fa[x]:
                fa[x] = getfa(fa[x])
            return fa[x]
        
        def union(x: int, y: int) -> None:
            f1 = getfa(x)
            f2 = getfa(y)
            if f1 != f2:
                fa[f1] = f2
                size[f2] += size[f1]
        
        def at(r: int, c: int) -> int:
            return r * n + c + 1
        
        ops = list(map(tuple, hits))
        isKicked = [False] * len(hits)
        for (i, (x, y)) in enumerate(ops):
            if grid[x][y] == 1:
                isKicked[i] = True
                grid[x][y] = 0

        for i in range(m):
            for j in range(n):
                if grid[i][j] == 1:
                    if i == 0:
                        union(at(i,j), 0)
                    elif grid[i-1][j] == 1:
                        union(at(i,j), at(i-1,j))
                    
                    if j > 0 and grid[i][j-1] == 1:
                        union(at(i, j), at(i, j-1))
        
        def ask():
            x = getfa(0)
            return size[x]
        
        ans = []
        cur = ask()
        for i in range(len(ops)-1, -1, -1):
            if not isKicked[i]:
                ans.append(0)
                continue

            (x, y) = ops[i]
            grid[x][y] = 1
            if x == 0:
                union(at(x, y), 0)
            elif grid[x-1][y] == 1:
                union(at(x, y), at(x-1, y))
            
            if x+1 < m and grid[x+1][y] == 1:
                union(at(x, y), at(x+1, y))
            
            if y > 0 and grid[x][y-1] == 1:
                union(at(x, y), at(x, y-1))
            
            if y+1 < n and grid[x][y+1] == 1:
                union(at(x, y), at(x, y+1))

            now = ask()
            # print(now, cur)
            ans.append(max(0, now - cur - 1))
            cur = now
        
        return list(reversed(ans))
```

