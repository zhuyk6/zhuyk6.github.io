---
title: Leetcode 410
categories:
  - 刷题
  - Leetcode
tags: [DP, 三分极值, 凸函数, 二分答案]
date: 2021-07-06 20:40:55
mathjax: true
---

[Leetcode 410](https://leetcode-cn.com/problems/split-array-largest-sum/)

<!--more-->

# Description

给定一个非负整数数组 `nums` 和一个整数 `m` ，你需要将这个数组分成 `m` 个非空的连续子数组。

设计一个算法使得这 `m` 个子数组各自和的最大值最小。

# Sample

## Sample 1：

```
输入：nums = [7,2,5,10,8], m = 2
输出：18
解释：
一共有四种方法将 nums 分割为 2 个子数组。 其中最好的方式是将其分为 [7,2,5] 和 [10,8] 。
因为此时这两个子数组各自的和的最大值为18，在所有情况中最小。
```

## Sample 2：

```
输入：nums = [1,2,3,4,5], m = 2
输出：9
```

## Sample 3：

```
输入：nums = [1,4,4], m = 3
输出：4
```

# Hint

- `1 <= nums.length <= 1000`
- `0 <= nums[i] <= 10^6`
- `1 <= m <= min(50, nums.length)`

# Solution

很经典的题目，也是Min-Max类型。

## DP

第一种思路，直接DP。很容易想到用 `f[i][j]` 表示 `nums[0]..nums[i]` 分成 `j` 组的答案，递归方程也很简单：

$$f[i][j] = \min_{k < i} \max \lbrace f[k][j-1], \sum_{p\in[k+1, i]}nums[p] \rbrace$$

很明显是一个 $O(N^2M)$ 的算法，不算优秀。能否优化？

- 首先不难发现，`f[*][j]` 只用到了 `f[*][j-1]` ，所以可以用滚动数组优化空间至 $O(N)$。而且可以把 `j` 的循环放在外层；
- 其次，计算连续和可以使用前缀数组优化至 $O(1)$ 。
- 关键还是时间复杂度，能否快速定位 `k` 的位置？是否具备单调性？
	- 仔细分析，这是一个 Min-Max 结构
	- 当 `k` 增大，`f[k][j-1]` 增大
	- 当 `k`	增大，`S[i] - S[k]` 减小
	- 所以 Min-Max 是一个下凸函数，可以用三分极值（或者二分交点）

## 二分答案

前面说了，这是个MIn-Max的结构，很容易想到能否**二分答案**：二分`Min = Limit`，然后判断能否在`Max <= Limit`的约束下存在可行解。

这也是二分答案的题目的显著特征，把优化问题转化为判定问题，前提当然是判定要比优化简单。

对于本题，判定可太简单了：尽可能让分组和大，但是不能超过LImit，看看最后会分成几组。

# Code

```cpp
class Solution {
public:
    int splitArray_DP(vector<int>& nums, int m) { // DP solution
        int n = nums.size();
        vector<int> sum(n);
        sum[0] = nums[0];
        for (int i = 1; i < n; ++i) sum[i] = sum[i-1] + nums[i];

        vector<vector<int>> f(n, vector<int>(m+1, 0x7fffffff));
        f[0][1] = nums[0];
        for (int i = 1; i < n; ++i) {
            f[i][1] = f[i-1][1] + nums[i]; 
        }
        
        for (int j = 2; j <= m; ++j) {
            for (int i = 0; i < n; ++i) {
                // int acc = nums[i];
                // for (int k = i-1; k >= 0; --k) {
                //     f[i][j] = min(f[i][j], max(f[k][j-1], acc));
                //     acc += nums[k];
                // }
                int L = 0, R = i, M;
                while (L < R) {
                    M = (L + R) >> 1;
                    if (f[M][j-1] <= sum[i] - sum[M]) {
                        L = M + 1;
                        f[i][j] = min(
                            f[i][j], 
                            max(f[M][j-1], sum[i] - sum[M]));
                    }
                    else {
                        R = M;
                    }
                }
                f[i][j] = min(
                    f[i][j], 
                    max(f[L][j-1], sum[i] - sum[L]));
            }
        }
        return f[n-1][m];
    }

    int splitArray(vector<int>& nums, int m) {	// Divide the answer
        auto judge = [&nums, m](int limit) {
            int ret = 1;
            int acc = 0;
            for (int v : nums) {
                if (acc + v <= limit) {
                    acc += v;
                }
                else {
                    acc = v;
                    ret += 1;
                }
            }
            // printf("limit=%d, groups=%d\n", limit, ret);
            return ret <= m;
        };
        int L = 0, R = 0, M;
        for (int v : nums) {
            R += v;
            L = max(L, v);
        }
        int ret = 0;
        while (L <= R) {
            M = (L + R) >> 1;
            if (judge(M)) {
                ret = M;
                R = M - 1;
            }
            else {
                L = M + 1;
            }
        }
        return ret;
    } 
};
```

