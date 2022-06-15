---
title: Leetcode 363
categories:
  - 刷题
  - Leetcode
tags: [前缀和, 二分]
date: 2021-07-05 20:39:16
mathjax: true
---

[Leetcode 363](https://leetcode-cn.com/problems/max-sum-of-rectangle-no-larger-than-k/)

<!--more-->

# Description

给你一个 `m x n` 的矩阵 `matrix` 和一个整数 `k` ，找出并返回矩阵内部矩形区域的不超过 `k` 的最大数值和。

题目数据保证总会存在一个数值和不超过 `k` 的矩形区域。

# Sample

![](https://assets.leetcode.com/uploads/2021/03/18/sum-grid.jpg)

```
输入：matrix = [[1,0,1],[0,-2,3]], k = 2
输出：2
解释：蓝色边框圈出来的矩形区域 [[0, 1], [-2, 3]] 的数值和是 2，且 2 是不超过 k 的最大数字（k = 2）。
```

# Hint

- `m == matrix.length`
- `n == matrix[i].length`
- `1 <= m, n <= 100`
- `-100 <= matrix[i][j] <= 100`
- `-10^5 <= k <= 10^5`


进阶：如果行数远大于列数，该如何设计解决方案？

# Solution

直接枚举矩阵 $O(N^4)$ ， 因为矩阵需要四条边来确定。

如果先枚举矩阵的上下边界（或者左右边界），那么就把二维矩阵转换为了一维数组，问题就变成了：给定一个数列 $\lbrace a_n\rbrace$ ，请你计算 

$$\max \lbrace \sum_{i\in[l, r]} a_i \leq K : l, r\in [1, n] \rbrace$$

这个问题就简单多了，通过前缀和 $S_i = \sum_{j\in [1, i]} a_j$ ，问题变成对于每个 $r$ ， 计算

$$\max \lbrace S_r - S_l \leq K : l < r \rbrace$$
$$S_r - K \leq S_l$$

所以只需要维护一个支持`add`、`lower_bound`的结构即可，选择`set`就行了。

整个算法的时间复杂度是 $O(M^2N\log N)$ 。如果矩阵行列差距过大，$M \gg N$，将矩阵转置即可。

# Code

```cpp
#include<bits/stdc++.h>
using namespace std;

class Solution {
public:
    int maxSumSubmatrix(vector<vector<int>>& matrix, int k) {
        int m = matrix.size();
        int n = matrix[0].size();
        if (m > n) {
            vector<vector<int>> mat(n, vector<int> (m));
            for (int i = 0; i < m; ++i)
                for (int j = 0; j < n; ++j)
                    mat[j][i] = matrix[i][j];
            return maxSumSubmatrix(mat, k);
        }

        vector<vector<int>> sum(m, vector<int>(n));
        sum[0][0] = matrix[0][0];
        for (int i = 1; i < m; ++i) 
            sum[i][0] = sum[i-1][0] + matrix[i][0];
        for (int j = 1; j < n; ++j)
            sum[0][j] = sum[0][j-1] + matrix[0][j];
        for (int i = 1; i < m; ++i)
            for (int j = 1; j < n; ++j)
                sum[i][j] = matrix[i][j]
                    + sum[i][j-1]
                    + sum[i-1][j]
                    - sum[i-1][j-1];

        auto get = [&sum](int a, int b, int c, int d) {
            return sum[c][d]
                - (a > 0 ? sum[a-1][d] : 0)
                - (b > 0 ? sum[c][b-1] : 0)
                + (a > 0 && b > 0 ? sum[a-1][b-1] : 0);
        };

        int ret = -0x7fffffff;
        for (int bot = 0; bot < m; ++bot)
            for (int top = bot; top < m; ++top) {
                auto S = [&get, bot, top](int i) {
                    return get(bot, 0, top, i);
                };
                set<int> s;
                s.insert(0);
                for (int i = 0; i < n; ++i) {
                    auto p = s.lower_bound(S(i) - k);
                    if (p != s.end()) {
                        ret = max(ret, S(i) - *p);
                    }
                    s.insert(S(i));
                }
            }

        return ret;
    }
};
```

