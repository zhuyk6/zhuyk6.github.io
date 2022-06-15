---
title: Leetcode 798
categories:
  - 刷题
  - Leetcode
tags: [差分]
date: 2021-09-15 07:24:52
mathjax: true
---

[Leetcode 798](https://leetcode-cn.com/problems/smallest-rotation-with-highest-score/)

<!--more-->

# Description

给定一个数组 `A`，我们可以将它按一个非负整数 `K` 进行轮调，这样可以使数组变为 `A[K], A[K+1], A{K+2], ... A[A.length - 1], A[0], A[1], ..., A[K-1]` 的形式。此后，任何 **值小于或等于其索引** 的项都可以记作一分。

例如，如果数组为 `[2, 4, 1, 3, 0]`，我们按 `K = 2` 进行轮调后，它将变成 `[1, 3, 0, 2, 4]`。这将记作 3 分，因为 `1 > 0` [no points], `3 > 1` [no points], `0 <= 2` [one point], `2 <= 3` [one point], `4 <= 4` [one point]。

在所有可能的轮调中，返回我们所能得到的最高分数对应的轮调索引 $K$。如果有多个答案，返回满足条件的最小的索引  $K$。

# Sample

**示例 1：**

```
输入：[2, 3, 1, 4, 0]
输出：3
解释：
下面列出了每个 K 的得分：
K = 0,  A = [2,3,1,4,0],    score 2
K = 1,  A = [3,1,4,0,2],    score 3
K = 2,  A = [1,4,0,2,3],    score 3
K = 3,  A = [4,0,2,3,1],    score 4
K = 4,  A = [0,2,3,1,4],    score 3
所以我们应当选择 K = 3，得分最高。
```

**示例 2：**

```
输入：[1, 3, 0, 2, 4]
输出：0
解释：
A 无论怎么变化总是有 3 分。
所以我们将选择最小的 K，即 0。
```

# Hint

- `A` 的长度最大为 `20000`。
- `A[i]` 的取值范围是 `[0, A.length]`。

# Solution

非常有趣的题目，有很多思考的角度，下面先说明一下我第一次想的方法。

## 直接询问

直接按照题目的要求，对 序列 `A` 和 下标序列 `I` 作差 得到序列 `D`，其中 $d_i = a_i - i$。这也就是 $K=0$ 时的差序列，随着 $K$ 的增大，我们不需要每次都重新作差：固定下标，值向左旋转；固定值，下标向右旋转。

$$\begin{matrix} 
    & a_0 & a_1 & a_2 & \cdots & a_{n-1} \\ 
k=0 & 0   & 1   & 2   & \cdots & n-1 \\
k=1 & n-1 & 0   & 1   & \cdots & n-2 \\
k=2 & n-2 & n-1 & 0   & \cdots & n-3 \\
\end{matrix}$$



很容易归纳出规律：

$$\begin{align}
d_i^0 & = a_i - i \\ 
d_i^k & = 
		\begin{cases}
        	d_i^{k-1} + 1, & \text{when $i \neq k-1$} \\
        	d_i^{k-1} + 1 - n & \text{when $i = k-1$}
		\end{cases} 
\end{align}$$



我们的任务是什么？对于每一个 $K$，询问有多少个 $d_i^k \leq 0$。

从任务可以看出来，我们甚至不需要迭代计算 $d_i^K$， 因为每次迭代 “几乎所有” $d_i^K = d_i^{K-1}+1$，我们改变询问即可：

- $k=0$， 询问多少个 $d_i \leq 0$
- $k=1$， 单独修改 $d_0$，询问多少个 $d_i \leq -1$
- $k=2$， 单独修改 $d_1$， 询问多少个 $d_i \leq -2$

这样一来，我们需要选取一个支持 `rank` 和 `insert` 、`delete` 的数据结构——选取平衡树（或jump list）即可。总的算法复杂度为 $O(N\log N)$。



## 答案贡献

第一种方法本质上还是在模拟旋转，然后不断统计答案的直接方法。

考虑一个“间接”问题：对于每一个 $a_i$， 哪些 $K$ 的旋转会使得 $a_i$ 参与贡献（答案统计）。

显然这个问题是可以 $O(1)$ 计算的：

1. 设 $x = a_i$ ， 旋转后的下标为 $p$
2. $a_i$ 参与贡献 当且仅当 $x \leq p$ ， 所以 $p = x, x+1, \ldots , n-1$
3. 旋转满足 $i - K \equiv p \mod n$，所以 $K \equiv i-p \mod n$
4. 最终计算得 $K \in [i - (n-1), i - x]$

> 需要说明，因为在 模 $N$ 意义下，所以 $K$ 所属的这个区间可能会 “裂开” 成两部分

现在问题就简单了，我们知道对于每一个 $a_i$ ，它参与贡献的答案区间，只需要选取合适的数据结构来不断的 “区间添加”即可。

线段树显然可以胜任。不过本题十分特殊，频繁的 添加操作，却只需要一次答案统计（全部询问），使用差分序列即可满足要求。



> 差分序列的特点是维护 $d_i = a_i - a_{i-1}$，对于区间 $[l, r]$ 整体改变 $\Delta$，仅需要 $d_l + \Delta$ 和 $d_{r+1} - \Delta$。然而差分序列的询问是非常致命的，$a_n = \sum_{i=0}^n d_i$。
>
> 差分序列的这个 “快速修改”、“慢询问”的特点，很难不让人联想到 Different List `DList :: [a] -> [a]`，这东西也是用于 快速“构建”列表的。其实某种意义上，差分序列也是一种特殊的 `DList`，只不过里面维护的不是 `[a]` 而是 `Array`。

# Code

```python
class Solution:
    def bestRotation(self, nums: List[int]) -> int:
        xs = nums
        n = len(xs)

        d = [0] * (n+1)
        
        def add(l, r, delta):
            d[l] += delta
            if r < n-1:
                d[r+1] -= delta

        for (i, x) in enumerate(xs):
            if x >= n:
                continue

            # p = x, x+1, x+2, ..., n-1
            if x > i:
                # print(i, x, "[{}, {}]".format(i+1, i-x+n))
                add(i + 1, i - x + n, 1)
            else:
                # print(i, x, "[{}, {}]".format(0, i-x))
                # k = i-x, i-(x+1), ..., i-i, i-(i+1)+n...
                add(0, i - x, 1)

                if i < n-1:
                    # print("[{}, {}]".format(i+1, n-1))
                    add(i + 1, n - 1, 1)
        
        ans = [0, 0]
        acc = d[-1]
        for i in range(n):
            acc += d[i]
            # print("k={}, score={}".format(i, acc))
            if acc > ans[0]:
                ans[0] = acc
                ans[1] = i
        return ans[1]
```

