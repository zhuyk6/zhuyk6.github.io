---
title: Leetcode 899
date: 2021-06-24 13:35:53
mathjax: true
categories:
- 刷题
- Leetcode
tags:

---

[Leetcode 899](https://leetcode-cn.com/problems/orderly-queue/)

<!--more-->

# Description

给出了一个由小写字母组成的字符串 `S`。然后，我们可以进行任意次数的移动。

在每次移动中，我们选择前`K` 个字母中的一个（从左侧开始），将其从原位置移除，并放置在字符串的末尾。

返回我们在任意次数的移动之后可以拥有的按字典顺序排列的最小字符串。

# Sample

## Sample 1

```
输入：S = "cba", K = 1
输出："acb"
解释：
在第一步中，我们将第一个字符（“c”）移动到最后，获得字符串 “bac”。
在第二步中，我们将第一个字符（“b”）移动到最后，获得最终结果 “acb”。
```

## Sample 2

```
输入：S = "baaca", K = 3
输出："aaabc"
解释：
在第一步中，我们将第一个字符（“b”）移动到最后，获得字符串 “aacab”。
在第二步中，我们将第三个字符（“c”）移动到最后，获得最终结果 “aaabc”。
```

# Hint

$1 \leq K \leq length(S) \leq 1000$

`S` 只由小写字母组成。

# Solution

这是非常有意思的一道题目，直觉告诉我们 `K` 是问题的关键，`K` 代表了可以操作的区间。

- $K=1$：没什么好说的，字符的相对位置是不会改变的，在循环队列里面找到字典序最小的即可。
- $K=2$：这就很有意思了，意味着我们可以交换两个相邻字符的顺序。
- $K> 2$：比$K=2$强，可以按照$K=2$操作。

具体来说，对于 $1\ldots i, i+1\ldots n$，可以先将$1\ldots i-1$按顺序移到后面，然后先$i+1$后$i$，再将$i+2\ldots n$依次操作，最终得到的序列是$1\ldots i-1, i+1, i, i+2, \ldots n$，也就是单独交换了$i, i+1$ 的顺序。

事实上，对于$K=2$，我们可以交换相邻字符的顺序，也就意味着可以交换任意两个字符的顺序，这是很显然的：对于$i\ldots j$ ，只需要将 $i$ 不断和后面的交换，然后将 $j$ 不断向前交换即可。

能够交换任意两个字符的顺序，也就一定可以得到字典序最小的结果。

> 其实“可以交换相邻字符”这一条件就够强了，因为这相当于“冒泡排序”。

# Code

```cpp
class Solution {
public:
    string orderlyQueue(string s, int k) {
        int n = s.length();
        if (k == 1) {
            auto cmp = [&](int x, int y) { // s[x..] < s[y..]
                for (int i = 0; i < n; ++i) {
                    if (s[(i+x)%n] != s[(i+y)%n])
                        return s[(i+x)%n] < s[(i+y)%n];
                }
                return false;
            };
            int ret = 0;
            for (int i = 1; i < n; ++i) {
                if (cmp(i, ret)) ret = i;
            }
            return s.substr(ret) + s.substr(0, ret);
        }
        else {
            for (int i = 0; i < n; ++i) {
                for (int j = 0; j < n-1; ++j) {
                    if (s[j] > s[j+1]) swap(s[j], s[j+1]);
                }
            }
            return s;
        }
    }
};
```