---
title: Leetcode 645
categories:
  - 刷题
  - Leetcode
tags: [位运算]
date: 2021-07-04 14:43:21
mathjax:
---

[Leetcode 645](https://leetcode-cn.com/problems/set-mismatch/)

<!--more-->

# Description

集合 `s` 包含从 `1` 到 `n` 的整数。不幸的是，因为数据错误，导致集合里面某一个数字复制了成了集合里面的另外一个数字的值，导致集合 **丢失了一个数字** 并且 **有一个数字重复** 。

给定一个数组 `nums` 代表了集合 `S` 发生错误后的结果。

请你找出重复出现的整数，再找到丢失的整数，将它们以数组的形式返回。

# Sample

## Sample 1

```
输入：nums = [1,2,2,4]
输出：[2,3]
```

## Sample 2

```
输入：nums = [1,1]
输出：[1,2]
```

# Hint

- `2 <= nums.length <= 10^4`
- `1 <= nums[i] <= 10^4`


# Solution

直觉告诉我们，肯定存在一个位运算的常数空间线性时间的算法。

首先，参照 “只有一个重复数字” 的解法，根据数字出现次数差异，尝试解决问题。

不妨设重复数字是`x`，丢失数字是`y`，在`S`中有`0`个`y`和`2`个`x`，剩下数字`1`次，很明显出现**奇偶差异**，我们使用**异或**可以分离。考虑到我们需要的是`x`和`y`，所以要把`S`和`1..n`放到一起异或，剩下的就是`x xor y`。

接下来怎么得到`x`和`y`是本题的重点：关键在于`x xor y`的含义，异或运算对于二进制**不同**的位结果为`1`。我们选择一位（不妨选择`lowbit`），根据这一位的差距，来将集合分成两类，显然`x`和`y`将被分到不同的类别中，接下来就简单了，对于每个类别，`x`或`y`都是其中奇偶性不同的那一个，继续用异或计算出即可。

# Code

```cpp
class Solution {
public:
    vector<int> findErrorNums(vector<int>& nums) {
        int n = nums.size();
        int xORy = 0;
        for (int v : nums) xORy ^= v;
        for (int i = 1; i <= n; ++i) xORy ^= i;
        int b = xORy & (-xORy);
        int x = 0, y = 0;
        for (int v : nums) {
            if (v & b)
                x ^= v;
            else
                y ^= v;
        }
        for (int i = 1; i <= n; ++i) {
            if (i & b)
                x ^= i;
            else
                y ^= i;
        }
        int ret1 = y, ret2 = x;
        for (int v : nums)
            if (v == x)
                ret1 = x, ret2 = y;
        return {ret1, ret2};
    }
};
```

