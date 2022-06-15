---
title: Leetcode 458
categories:
  - 刷题
  - Leetcode
tags: [信息论]
date: 2021-07-13 21:27:16
mathjax: true
---

[Leetcode 458](https://leetcode-cn.com/problems/poor-pigs/)

<!--more-->

# Description

有 `buckets` 桶液体，其中 **正好** 有一桶含有毒药，其余装的都是水。它们从外观看起来都一样。为了弄清楚哪只水桶含有毒药，你可以喂一些猪喝，通过观察猪是否会死进行判断。不幸的是，你只有 `minutesToTest` 分钟时间来确定哪桶液体是有毒的。

喂猪的规则如下：

1. 选择若干活猪进行喂养
2. 可以允许小猪同时饮用任意数量的桶中的水，并且该过程不需要时间。
3. 小猪喝完水后，必须有 `minutesToDie` 分钟的冷却时间。在这段时间里，你只能观察，而不允许继续喂猪。
4. 过了 `minutesToDie` 分钟后，所有喝到毒药的猪都会死去，其他所有猪都会活下来。
5. 重复这一过程，直到时间用完。

给你桶的数目 `buckets` ，`minutesToDie` 和 `minutesToTest` ，返回在规定时间内判断哪个桶有毒所需的 **最小** 猪数。

# Sample

**示例 1：**

```
输入：buckets = 1000, minutesToDie = 15, minutesToTest = 60
输出：5
```


**示例 2：**

```
输入：buckets = 4, minutesToDie = 15, minutesToTest = 15
输出：2
```

**示例 3：**
```
输入：buckets = 4, minutesToDie = 15, minutesToTest = 30
输出：2
```

# Hint

- `1 <= buckets <= 1000`
- `1 <= minutesToDie <= minutesToTest <= 100`

# Solution

非常经典的一道信息论的题目，与之类似的，还有残次品称重等等。

首先把问题转化为，`m` 个桶，`s = minutesToTest / minutesToDie` 轮测试，最少需要多少只猪。

考虑对偶问题，`n`只猪，`s`轮测试，最多测试多少个桶。

先看简单的情况，对于`1`只猪，`s`轮测试，能测试多少个桶？显然是`s+1`个桶，因为最坏的情况，`s`轮都没有死，那剩下的就是有毒的。

那`2`只猪呢，我们考虑将桶摆成二维平面，那么可以测试 $(s+1)^2$ 个桶。具体方法：第一只猪一行一行的测试，每次测试饮用该行的所有桶混合；第二只猪一列一列测试，饮用该列所有桶的混合。显然 $s$ 轮测试下来，在二维平面必有唯一交点。

`3` 只猪就摆成三维正方体，$n$ 只就摆成 $n$ 维正立方体，可以测试 $(s+1)^n$ 个桶。

换一种思路，用编码的思想：一只猪就是一位数字，最多编码 $s+1$ 种状态，所以 $n$ 只猪可以编码  $(s+1)^n$ 种状态。假设 $x$ 表示第 $x$ 个桶有毒药（从0计数），将 $x$ 用 $s+1$ 进制表示：

$$x = a_0 + (s+1)\times a_1 + (s+1)^2 \times a_2 + \cdots $$

第 `k` 只猪，就测试 $a_k$ 的值，具体方法，第 $i$ 轮，将所有 $s+1$ 进制下，$a_k = i$ 的桶混合饮用，显然 $s$ 轮下来，可以确定 $a_k$ 的值。

通过上述两种方法，我们找到了测试 $(s+1)^n$ 个桶的方法，但是这是否是最优的？

证明最优，肯定需要用**信息论**：

对于一只猪，最多测试`s+1`个桶，获取到的信息量是 $\log_2(s+1)$ bit，所以`n`只猪最多得到的信息量是 $n \times \log_2(s+1)$ bit，因为总共有 $m$ 个桶，所以我们需要的信息量是 $\log_2 m$ bit，想要能够测试成功，需要测试获得的信息量大于问题需要的信息量：

$$
\begin{align}
n \times \log_2(s+1) & \geq \log_2 m \\ 
\log_2 (s+1)^n & \geq \log_2 m \\
(s+1)^n & \geq m
\end{align}
$$

所以`n`只猪的测试极限就是 $(s+1)^n$，我们上述方法达到了极限。

# Code

```python Python code
import math
class Solution:
    def poorPigs(self, buckets: int, minutesToDie: int, minutesToTest: int) -> int:
        s = minutesToTest // minutesToDie
        m = buckets
        # (s+1)^n >= M
        n = math.ceil(math.log(m, s+1))
        return n
```