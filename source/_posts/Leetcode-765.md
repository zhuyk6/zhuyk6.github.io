---
title: Leetcode 765
categories:
  - 刷题
  - Leetcode
tags: [并查集]
date: 2021-08-30 03:07:54
mathjax: false
---

[Leetcode 765](https://leetcode-cn.com/problems/couples-holding-hands/)

<!--more-->

# Description

`N` 对情侣坐在连续排列的 `2N` 个座位上，想要牵到对方的手。 计算最少交换座位的次数，以便每对情侣可以并肩坐在一起。 一次交换可选择任意两人，让他们站起来交换座位。

人和座位用 `0` 到 `2N-1` 的整数表示，情侣们按顺序编号，第一对是 `(0, 1)`，第二对是 `(2, 3)`，以此类推，最后一对是 `(2N-2, 2N-1)`。

这些情侣的初始座位 `row[i]` 是由最初始坐在第 `i` 个座位上的人决定的。

# Sample

``` sample 1:
输入: row = [0, 2, 1, 3]
输出: 1
解释: 我们只需要交换row[1]和row[2]的位置即可。
```

``` sample 2:
输入: row = [3, 2, 0, 1]
输出: 0
解释: 无需交换座位，所有的情侣都已经可以手牵手了。
```

# Hint

1. `len(row)` 是偶数且数值在 `[4, 60]` 范围内。
2. 可以保证 `row` 是序列 `0...len(row)-1` 的一个全排列。

# Solution

很容易被这道题目给虎住，不清楚怎么下手，但其实只是纸老虎。

表面上有 `2n` 个位置，其实只有 `n` 个“盒子”：考虑最终状态，所有情侣配对完成，每一对情侣一定占据一个盒子——0-1,2-3,4-5……注意，如果一对情侣不在一个盒子，即使相邻也不行，还是要交换（因为你不交换影响别人配对）。

考虑 `n` 个结点的图，如果初始情侣 `x` 和 `y` 在一个盒子，那么必须经过交换，将 `x` 与 `y` 之间连一条边。我们的问题就是对于一个连通块，最少交换是多少？显然对于一个大小为 `m` 的连通块，交换次数为 `m-1` ：每次交换使得一个情侣配对。

所以算法就很简单了，找到这张图所有的连通块，计算连通块的大小，统计答案。题目只需要连通信息，所以可以使用 并查集 。

# Code

```python
class Solution:
    def minSwapsCouples(self, row: List[int]) -> int:
        n = len(row) >> 1
        fa = [i for i in range(n)]

        def getfa(x: int) -> int:
            if x != fa[x]:
                fa[x] = getfa(fa[x])
            return fa[x]
        
        def union(x: int, y: int) -> None:
            f1 = getfa(x)
            f2 = getfa(y)
            fa[f1] = f2
        
        i = 0
        while i < 2 * n:
            x = row[i] // 2
            y = row[i+1] // 2
            union(x, y)
            i += 2
        
        cnt = dict()
        for i in range(n):
            x = getfa(i)
            if x in cnt:
                cnt[x] += 1
            else:
                cnt[x] = 1
        ret = 0
        for k, v in cnt.items():
            ret += v - 1
        return ret
```
