---
title: Leetcode 459
categories:
  - 刷题
  - Leetcode
tags: [KMP, 字符串]
date: 2022-02-22 20:38:33
mathjax: true
---

[Leetcode 459](https://leetcode-cn.com/problems/repeated-substring-pattern/)

<!--more-->

# Description

给定一个非空的字符串 s ，检查是否可以通过由它的一个子串重复多次构成。


# Sample
 
示例 1:

```
输入: s = "abab"
输出: true
解释: 可由子串 "ab" 重复两次构成。
```

示例 2:
```
输入: s = "aba"
输出: false
```

示例 3:

```
输入: s = "abcabcabcabc"
输出: true
解释: 可由子串 "abc" 重复四次构成。 (或子串 "abcabc" 重复两次构成。)
```

# Hint

- $1 \leq s.length \leq 10^4$
- s 由小写英文字母组成

# Solution

本题很有意思，非常值的思考。

首先我们来考虑必要性，如果 $s = p+p+p+\cdots +p$，那么显然将 $s$ 右移 $|p|$ 位一定是可以和原来的 $s$ 对齐的，如下表：

|   | 0 | 1 | 2 | 3 |
|---|---|---|---|---|
| s | p | p | p |   |
| s'|   | p | p | p |

换言之，如果另 $t = s + s$，进行 `findall(t, s)` 算法求解所有 $s$ 在 $t$ 中出现的位置，结果除了 $0, N$ 以外一定包含其他位置。

这个条件是充分的吗？

设 $0 < k < N$ 是 `findall(t, s)` 的结果之一，意味着 $s$ 右移 $k$ 位可以和原来重合，不难观察发现有下面的关系：

$$\forall i, \quad s[i] = s[i+k] = s[i+2k] = \cdots = s[i+nk] \quad , \mod N$$

在 $\mod N$ 意义下，$s$ 有长度为 $k$ 的循环节，也就是说 $s$ 有长度为 $\gcd(N, k)$ 的循环节，又因为 $\gcd(N, k) | N$ 且 $k < N$，所以我们找到了 $p$ 长度为 $\gcd(N, k)$，充分性证毕。

# Code

```cpp
int kmp(const string & s, const string & p) {
    // get next
    vector<int> next(p.length()+1);
    next[0] = -1;
    int j = -1;
    for (int i = 0; i < p.length(); ++i) {
        while (j != -1 && p[j] != p[i]) j = next[j];
        ++j;
        next[i+1] = p[i+1] == p[j] ? next[j] : j;
    }

    // get index
    j = 0;
    for (int i = 0; i < s.length(); ++i) {
        while (j != -1 && p[j] != s[i]) j = next[j];
        ++j;
        if (j == p.length())
            return i - j + 1;
    }
    return -1;
}


class Solution {
public:
    bool repeatedSubstringPattern(string s) {
        string t = s + s;
        t = t.substr(1, 2 * s.length() - 2);
        // cout << t << endl;
        // cout << s << endl;
        return kmp(t, s) != -1;
    }
};
```