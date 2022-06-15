---
title: Leetcode 850
categories:
  - 刷题
  - Leetcode
tags: [离散化, 扫描线, 线段树]
date: 2021-09-05 21:08:00
mathjax: true
---

[Leetcode 850](https://leetcode-cn.com/problems/rectangle-area-ii/)

<!--more-->

# Description

我们给出了一个（轴对齐的）矩形列表 `rectangles` 。 对于 `rectangle[i] = [x1, y1, x2, y2]`，其中 `(x1，y1)` 是矩形 `i` 左下角的坐标，`(x2，y2)` 是该矩形右上角的坐标。

找出平面中所有矩形叠加覆盖后的总面积。 由于答案可能太大，请返回它对 `10^9+7` 取模的结果。

![](sample.png)

# Sample

```
输入：[[0,0,2,2],[1,0,2,3],[1,0,3,1]]
输出：6
解释：如图所示。
```


```
输入：[[0,0,1000000000,1000000000]]
输出：49
解释：答案是 10^18 对 (10^9 + 7) 取模的结果， 即 (10^9)^2 → (-7)^2 = 49 。
```

# Hint

提示：

- `1 <= rectangles.length <= 200`
- `rectanges[i].length = 4`
- `0 <= rectangles[i][j] <= 10^9`
- 矩形叠加覆盖后的总面积不会超越 `2^63 - 1` ，这意味着可以用一个 `64` 位有符号整数来保存面积结果。

# Solution

非常经典的题目，在这里一步步分析。

## 离散化

因为难点在于矩阵会覆盖、重叠，所以很自然想到维护一个标记数组（或者一个`set`），每次将一个矩阵中的所有 $1\times 1$ 的块标记，最后统计有多少个 $1\times 1$ 的方块即可。

这样做显然正确，但是矩阵可能很大 $10^9$，不过矩阵的数目显然很小，矩阵只有 $O(N)$ ，所以可以采用 **离散化** 的思路，将矩阵坐标重新映射，这样坐标就只有 $O(2N)$ ，使用暴力标记的方法，算法复杂度为 $O(N^3)$。

需要指出，因为坐标被重映射了，所以统计答案时，需要映射回原来的坐标。



## 扫描线

暴力的方法 $O(N^3)$ 并没有让人满意，之所以这么慢，是因为没有充分考虑 **矩阵** 的性质，只是把矩阵当作一般的图形。那么矩阵有什么性质呢？横平竖直的矩阵是严格被四条直线约束的。

换句话说，如果我们沿着 $x$ 轴从小到大扫描，遇到一个矩阵的左边界，那么直到它的右边界为止，这个矩阵上下边界中间夹着的区域都参与答案的贡献。

这就是扫描线算法，我们将一个矩阵拆成两部分 `(Add, x1, y1, y2)` 和 `(Del, x2, y1, y2)`，对 $x$ 进行排序，然后从小到大扫描，只需要维护一个数据结构，支持线段的添加和删除，并统计线段的总长度（考虑覆盖）。

扫描线算法的本质，其实就是从 $x$ 轴上的每一个点，向 $y$ 轴正方向看过去，看到的参与答案统计，本质就是微积分：

$$\int_{-\infty}^{+\infty} y(x) \text{d}x$$



按照扫描线模型，我们如果选取普通数组作为数据结构，维护统计 $y$ 轴的线段，将 $y$ 轴离散化，显然每次添加、删除、统计的复杂度都是 $O(N)$ ，所以算法整体的复杂度为 $O(N^2)$。



## 线段树

显然普通数组太拉跨了，现在的问题已经变成如何选取合适的数据结构，支持 区间添加、区间删除、整体统计（考虑覆盖：正数对答案只贡献一次）。

直接上线段树就OK了，也不用实现lazy标记，因为该问题太过于特殊，删除一条线段一定建立在之前添加过这条线段的基础上，所以压根不需要将覆盖的标记向下传递。



# Code

```python
#
# @lc app=leetcode.cn id=850 lang=python3
#
# [850] 矩形面积 II
#

# @lc code=start
from typing import *

class Node:
    def __init__(self, l, r, arr) -> None:
        self.l = l
        self.r = r
        self.cnt = 0
        self.ans = 0
        self.arr = arr

        self.left = None
        self.right = None
        if l < r:
            m = (l + r) >> 1
            self.left = Node(l, m, arr)
            self.right = Node(m+1, r, arr)
        
    def modify(self, L, R, delta):
        if L <= self.l and self.r <= R:
            self.cnt += delta
        else:
            m = (self.l + self.r) >> 1
            if L <= m:
                self.left.modify(L, R, delta)
            if R > m:
                self.right.modify(L, R, delta)
        
        if self.cnt > 0:
            self.ans = self.arr[self.r+1] - self.arr[self.l]
        else:
            if self.l == self.r:
                self.ans = 0
            else:
                self.ans = self.left.ans + self.right.ans

class Segment:
    def __init__(self, xs) -> None:
        ys = sorted(set(xs))
        d = dict()
        for i in range(len(ys)):
            d[ys[i]] = i
        self.num2pos = d
        self.pos2num = ys
        # self.seg = [0 for i in range(len(ys))]
        self.seg = Node(0, len(ys)-1, ys)

    def count(self) -> int:
        # acc = 0
        # for i in range(len(self.seg)):
        #     if self.seg[i] > 0:
        #         acc += self.pos2num[i+1] - self.pos2num[i]
        # return acc
        return self.seg.ans

    def add(self, l: int, r: int, delta: int)->None:
        L = self.num2pos[l]
        R = self.num2pos[r]
        # for i in range(L, R):
        #     self.seg[i] += delta
        self.seg.modify(L, R-1, delta)
        

class Solution:
    def rectangleArea(self, rectangles: List[List[int]]) -> int:
        ops = []
        for rec in rectangles:
            ops.append((rec[0], "A", rec[1], rec[3]))
            ops.append((rec[2], "D", rec[1], rec[3]))
        
        ys = []
        for rec in rectangles:
            ys.append(rec[1])
            ys.append(rec[3])
        
        ops.sort()
        ans = 0
        seg = Segment(ys)
        for i in range(len(ops)):
            # print("op={}".format(ops[i]))
            if i > 0 and ops[i][0] > ops[i-1][0]:
                # print("cnt={}".format(seg.count()))
                ans += seg.count() * (ops[i][0] - ops[i-1][0])
            # print("ans={}".format(ans))
            (x, ty, y1, y2) = ops[i]
            if ty == "A":
                seg.add(y1, y2, 1)
            else:
                seg.add(y1, y2, -1)
        
        return ans % int(1e9+7) 

# recs = [[25,20,70,27],[68,80,79,100],[37,41,66,76]]
# print(Solution().rectangleArea(recs))
```

