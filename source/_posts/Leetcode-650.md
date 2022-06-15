---
title: Leetcode 650
date: 2021-06-05 17:04:23
mathjax: true
categories:
- 刷题
- Leetcode
tags:

---

[Leetcode 650](https://leetcode-cn.com/problems/2-keys-keyboard/)

<!--more-->

## Description

最初在一个记事本上只有一个字符 `'A'`。你每次可以对这个记事本进行两种操作：

- Copy All (复制全部) : 你可以复制这个记事本中的所有字符(部分的复制是不允许的)。
- Paste (粘贴) : 你可以粘贴你上一次复制的字符。

给定一个数字 $n$ 。你需要使用最少的操作次数，在记事本中打印出恰好 $n$ 个 `'A'`。输出能够打印出 $n$ 个 `'A'` 的最少操作次数。

## Sample

### Sample 1
```
输入: 3
输出: 3
解释:
最初, 我们只有一个字符 'A'。
第 1 步, 我们使用 Copy All 操作。
第 2 步, 我们使用 Paste 操作来获得 'AA'。
第 3 步, 我们使用 Paste 操作来获得 'AAA'。
```

## Hint

$n$ 的取值范围是 $[1, 1000]$ 。

## Solution

### DP

DP的思路是很直观的，$f[i][j]$，其中 $i$ 表示现在有的字符数目，$j$ 表示clipboard剪切板的字符数目。

状态转移也很简单：

- Copy ： $f[i][i] = \min \lbrace f[i][j] + 1 : j\in [0, i)\rbrace$
- Paste ： $f[i][j] = f[i-j][j] + 1$

时间复杂度和空间复杂度都是 $O(N^2)$ 。

### Math

这道题仔细分析其实很有意思，很显然剪切板clipboard的大小是单调增长的。

整个操作序列`CPPCPCPPPCPP...`，我们将操作序列按照`C`来分段，这么做很trivial。

- 第一段`CPP...`之后，经过了 $p_1$ 次操作，其中一次`C`和 $p_1-1$ 次`P`，此时序列长度为 $p_1$ ；
- 第二段`CPP...`之后，经过了 $p_2$ 次操作，此时序列长度为 $p_1\times p_2$ ；

那么很显然 $n = p_1\times p_2\times \cdots \times p_m$ ，也就是对 $n$ 进行因式分解。

下面我们要证明，分解要比不分解要优：

假设 $n=p\times q$ ，如果不分解，需要 $n=p\times q$ 次操作，如果分解，需要 $p+q$ 次操作，对于 $p,q \geq2$ ，很显然 $p+q \leq p\times q$ 。

综上所述，本题只需要对 $n$ 进行质因数分解 $n=p_1^{q_1}p_2^{q_2}\cdots p_m^{q_m}$，答案就是 $\sum p_iq_i$ 。

## Code

```cpp

class Solution {
public:
    int minSteps_v1(int n) {
        vector<vector<int> > f(n+1, vector<int> (n+1, 1e9));
        f[1][0] = 0;
        for (int i = 1; i <= n; ++i) {
            for (int j = 0; j < i; ++j) {
                f[i][i] = min(f[i][i], f[i][j] + 1);
                if (i + j <= n) {
                    f[i + j][j] = min(f[i+j][j], f[i][j]+1);
                }
            }
            if (i + i <= n) {
                f[i + i][i] = min(f[i + i][i], f[i][i] + 1);
            }
        }
        return *min_element(f[n].begin(), f[n].end());
    }

    int minSteps(int n) {
        int ret = 0;
        for (int d = 2; d <= n; ++d) {
            while (n % d == 0) {
                ret += d;
                n /= d;
            }
        }
        return ret;
    }
};
```