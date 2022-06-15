---
title: Leetcode 390
categories:
  - 刷题
  - Leetcode
tags: [Math]
date: 2021-06-27 13:27:27
mathjax: true
---

[Leetcode 390](https://leetcode-cn.com/problems/elimination-game/)

<!--more-->

# Description

给定一个从 $1$ 到 $n$ 排序的整数列表。

- 首先，从左到右，从第一个数字开始，每隔一个数字进行删除，直到列表的末尾。
- 第二步，在剩下的数字中，从右到左，从倒数第一个数字开始，每隔一个数字进行删除，直到列表开头。

我们不断重复这两步，从左到右和从右到左交替进行，直到只剩下一个数字。返回长度为 $n$ 的列表中，最后剩下的数字。

# Sample

```
输入
n = 9
1 2 3 4 5 6 7 8 9
2 4 6 8
2 6
6

输出:
6
```

# Solution

很有意思的一道数学题，直接模拟肯定凉凉，但是可以通过模拟来找规律：

- $n \equiv 0 \quad(\mod 2)$ 
  - Left to right：`1 2 3 4`$\rightarrow$ `_ 2 _ 4`
  - Right to left：`1 2 3 4`$\rightarrow$ `1 _ 3 _`
- $n \equiv 1 \quad (\mod 2)$
  - Left to right: `1 2 3 4 5` $\rightarrow$ `_ 2 _ 4 _`
  - Right to left: `1 2 3 4 5` $\rightarrow$ `_ 2 _ 4 _`

我们可以发现非常明显的规律，问题是如何构造递归？

本题的关键在于，无论怎么变换，数列都是**等差数列**，所以我们只需要用`s`和`d`分别表示首项和公差即可。

# Code

```cpp
class Solution {
public:
    int lastRemaining(int n) {
        function<int(int, int, int)> LR, RL;
        LR = [&](int x, int s, int d) {
            if (x == 1) return s;
            return RL(x >> 1, s + d, d << 1);
        };
        RL = [&](int x, int s, int d) {
            if (x == 1) return s;
            if (x & 1)
                return LR(x >> 1, s + d, d << 1);
            else
                return LR(x >> 1, s, d << 1);
        };
        return LR(n, 1, 1);
    }
};
```



