---
title: Leetcode 936
categories:
  - 刷题
  - Leetcode
tags: [拓扑排序]
date: 2021-09-14 05:55:48
mathjax: true
---

[Leetcode 936](https://leetcode-cn.com/problems/stamping-the-sequence/description/)

<!--more-->

# Description

你想要用**小写字母**组成一个目标字符串 `target`。 

开始的时候，序列由 `target.length` 个 `'?'` 记号组成。而你有一个小写字母印章 `stamp`。

在每个回合，你可以将印章放在序列上，并将序列中的每个字母替换为印章上的相应字母。你最多可以进行 `10 * target.length` 个回合。

举个例子，如果初始序列为 `"?????"`，而你的印章 `stamp` 是 `"abc"`，那么在第一回合，你可以得到 `"abc??"`、`"?abc?"`、`"??abc"`。（请注意，印章必须完全包含在序列的边界内才能盖下去。）

如果可以印出序列，那么返回一个数组，该数组由每个回合中被印下的最左边字母的索引组成。如果不能印出序列，就返回一个空数组。

例如，如果序列是 `"ababc"`，印章是 `"abc"`，那么我们就可以返回与操作 `"?????"` -> `"abc??"` -> `"ababc"` 相对应的答案 `[0, 2]`；

另外，如果可以印出序列，那么需要保证可以在 `10 * target.length` 个回合内完成。任何超过此数字的答案将不被接受。

# Sample

**示例 1：**

```
输入：stamp = "abc", target = "ababc"
输出：[0,2]
（[1,0,2] 以及其他一些可能的结果也将作为答案被接受）
```

**示例 2：**

```
输入：stamp = "abca", target = "aabcaca"
输出：[3,0,1]
```

# Hint

1. `1 <= stamp.length <= target.length <= 1000`
2. `stamp` 和 `target` 只包含小写字母。

# Solution

首先，不难想到 “倒过来” 处理：从 `target` 反向还原为 `"????..???"`。很容易发现，这就是简单的字符串匹配，特别的 `'?'` 可以匹配任意字符。

不难证明，“匹配就还原”的贪心策略是正确的——因为还原使得字符串变得更“优”了，更容易匹配了，并且由于只要求给出可行的解，所以可以采用贪心策略（如果要求最小操作，那就麻烦了）。

由于“匹配就还原”，所以不难想出一种 $O(N^3)$ 的暴力匹配方法。

官方解答很巧妙：考虑每一个可能作为“还原”起点的位置 `p`，能够在 `p` 处还原 **当且仅当** `t[p:p+m] == s` ，这里的 `==` 允许 `'?'` 匹配任意字符。那么对于任意的位置 `p` ，我们可以维护一个集合 `todo[p]` ，`todo[p]` 表示那些 **不匹配** 的位置。显然，当且仅当 `todo[p]` 为空集，才允许从 `p` 处还原。

有了这么个思路，我们就可以倒过来还原，不断的将 `todo[p]` 为空集的 `t[p:p+m]` 全部置为 `'?'`。每当一个位置 `q` 变为 `'?'`，我们就将 `q` 从所有的 `todo[p]` 中去除。

到了这里，我们不难发现，其实这就是 **拓扑排序**，一种变形的拓扑排序。

我们将每个位置 `p` 抽象成一个结点，`t[p:p+m]` 就是对 `p` 的约束：具体来说，如果 `t[p+k] != s[k]`，那么就从 `p+k` 连一条有向边到 `p`。然后进行拓扑排序，不同的是，当从图中删除结点 `p` 时，顺带将 `p..p+m-1` 都一并删除。

这是非常巧妙的用DAG表示约束建模，拓扑排序的时间复杂度是 $O(N+M) = O(N^2)$。

# Code

```python
from typing import *
import queue

class Solution:
    def movesToStamp(self, stamp: str, target: str) -> List[int]:
        s, t = stamp, target
        m, n = len(s), len(t)

        to = [[] for _ in range(n)]
        deg = [0] * (n - m + 1)
        done = [False] * n

        for i in range(n - m + 1):
            for j in range(m):
                if t[i + j] != s[j]:
                    to[i + j].append(i)
                    deg[i] += 1
        
        q = queue.Queue()
        for i in range(n-m+1):
            if deg[i] == 0:
                q.put(i)
        
        ans = []
        while not q.empty():
            x = q.get()
            ans.append(x)
            for i in range(m):
                y = x + i
                if not done[y]:
                    done[y] = True
                    for z in to[y]:
                        deg[z] -= 1
                        if deg[z] == 0:
                            q.put(z)
        
        if all(done):
            return ans[::-1]
        else:
            return []
```

