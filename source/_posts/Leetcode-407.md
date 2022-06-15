---
title: Leetcode 407
categories:
  - 刷题
  - Leetcode
tags: [BFS, Heap]
date: 2021-07-05 23:26:03
mathjax:
---

[Leetcode 407](https://leetcode-cn.com/problems/trapping-rain-water-ii/)

<!--more-->

# Description

给你一个 `m x n` 的矩阵，其中的值均为非负整数，代表二维高度图每个单元的高度，请计算图中形状最多能接多少体积的雨水。

# Sample

```
给出如下 3x6 的高度图:
[
  [1,4,3,1,3,2],
  [3,2,1,3,2,4],
  [2,3,3,2,3,1]
]

返回 4 。
```

![](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/rainwater_empty.png)

如上图所示，这是下雨前的高度图的状态。

![](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/rainwater_fill.png)

下雨后，雨水将会被存储在这些方块中。总的接雨水量是4。

# Hint

- `1 <= m, n <= 110`
- `0 <= heightMap[i][j] <= 20000`


# Solution

非常有趣的一道题目，想法很多，但不知道如何入手。

起初我的想法是，给所有点按照高度从小到大操作，选取最小的，然后灌水至和周围一样高，这需要一个堆来维护。但是问题在于，如果有多个一样高的最小值相邻，怎么搞？大概需要图论，将之看作一个连通块，求这个连通块的相邻最小高度，然后统一灌水。那堆中就不能存储单个点了，直接存储连通块。这样做应该是可以的，不过有点复杂。

接下来我想出了一个错误的方法：对于每个点，它能灌多少水只取决于四个方向的最大值中的最小值。
为什么这个方法错误呢，很简单，并不是只取决于这四个方向，可能灌水有折线。

其实标准答案的思想很简单：“**木桶原理**”。对于一个木桶，能灌多少水取决于最短的边框。

首先将最外围一圈当作边框，然后找到最短边框，尝试灌水：如果相邻比边框小，灌水至边框高度，然后将之当作新的边框。

这个方法的魅力在于，边框由外向内的不断迭代，每次边框的改变，顺便灌水。非常漂亮。

# Code

```cpp
class Solution {
public:
    int trapRainWater(vector<vector<int>>& heightMap) {
        int m = heightMap.size();
        int n = heightMap[0].size();

        if (m < 3 || n < 3) return false;
        
        vector<vector<bool>> vis(m, vector<bool>(n, false));
        priority_queue<pair<int, pair<int, int>>, 
            vector<pair<int, pair<int, int>>>, 
            greater<pair<int, pair<int, int> > > > q;
        for (int i = 0; i < m; ++i) {
            vis[i][0] = true;
            vis[i][n-1] = true;
            q.push(make_pair(heightMap[i][0], 
                            make_pair(i, 0)));
            q.push(make_pair(heightMap[i][n-1], 
                            make_pair(i, n-1)));
        }
        for (int j = 0; j < n; ++j) {
            vis[0][j] = true;
            vis[m-1][j] = true;
            q.push(make_pair(heightMap[0][j], 
                            make_pair(0, j)));
            q.push(make_pair(heightMap[m-1][j], 
                            make_pair(m-1, j)));
        }
        
        const vector<pair<int, int>> directions = {
            {-1, 0}, {1, 0}, {0, -1}, {0, 1}
        };
        auto check = [m, n](int x, int y) {
            return 0 <= x && x < m && 0 <= y && y < n;
        };

        int ret = 0;
        while (!q.empty()) {
            auto tmp = q.top();
            q.pop();
            int h = tmp.first;
            int x = tmp.second.first;
            int y = tmp.second.second;
            for (auto par : directions) {
                int dx = par.first;
                int dy = par.second;
                if (check(x+dx, y+dy) && !vis[x+dx][y+dy]) {
                    vis[x+dx][y+dy] = true;
                    if (heightMap[x+dx][y+dy] < h) {
                        ret += h - heightMap[x+dx][y+dy];
                        q.push(make_pair(h, 
                                make_pair(x+dx, y+dy)));
                    }
                    else {
                        q.push(make_pair(heightMap[x+dx][y+dy], 
                                make_pair(x+dx, y+dy)));
                    }
                }
            }
        }
        return ret;
        
    }
};
```