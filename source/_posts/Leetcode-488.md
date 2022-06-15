---
title: Leetcode 488
categories:
  - 刷题
  - Leetcode
tags: [DFS, BFS, 剪枝]
date: 2021-07-13 01:35:48
mathjax:
---

[Leetcode 488](https://leetcode-cn.com/problems/zuma-game/)

<!--more-->

# Description

回忆一下祖玛游戏。现在桌上有一串球，颜色有红色(R)，黄色(Y)，蓝色(B)，绿色(G)，还有白色(W)。 现在你手里也有几个球。

每一次，你可以从手里的球选一个，然后把这个球插入到一串球中的某个位置上（包括最左端，最右端）。接着，如果有出现三个或者三个以上颜色相同的球相连的话，就把它们移除掉。重复这一步骤直到桌上所有的球都被移除。

找到插入并可以移除掉桌上所有球所需的最少的球数。如果不能移除桌上所有的球，输出 `-1` 。

# Sample

**示例 1：**

```
输入：board = "WRRBBW", hand = "RB"
输出：-1
解释：WRRBBW -> WRR[R]BBW -> WBBW -> WBB[B]W -> WW
```

**示例 2：**

```
输入：board = "WWRRBBWW", hand = "WRBRW"
输出：2
解释：WWRRBBWW -> WWRR[R]BBWW -> WWBBWW -> WWBB[B]WW -> WWWW -> empty
```

**示例 3：**

```
输入：board = "G", hand = "GGGGG"
输出：2
解释：G -> G[G] -> GG[G] -> empty 
```

**示例 4：**

```
输入：board = "RBYYBBRRB", hand = "YRBGB"
输出：3
解释：RBYYBBRRB -> RBYY[Y]BBRRB -> RBBBRRB -> RRRB -> B -> B[B] -> BB[B] -> empty 
```

# Hint

- 你可以假设桌上一开始的球中，不会有三个及三个以上颜色相同且连着的球。
- `1 <= board.length <= 16`
- `1 <= hand.length <= 5`
- 输入的两个字符串均为非空字符串，且只包含字符 `'R','Y','B','G','W'`。

# Solution

非常明显的一道搜索题目。

如果直接暴力搜索的话，枚举每一个球的插入位置，具体复杂度就不分析了，因为涉及增减，总之复杂度是很高的。

肯定要剪枝。

一种错误的剪枝：每次添加球一定是为了达成“消减”。所以每次只在连续的2个球添加。

为什么说是错误的呢，示例3就是，有可能某种颜色只有一个。

改进的剪枝：每次添加球，其左右位置一定有一样的颜色。

这样依然是错误的。反例并不容易想到，测试数据中有一组数据：

```
board = "RRWWRRYYRR"
hand = "WY"
```

这组数据很容易想到先消除`'W'`，但是相邻的`R`就跟着消除了，这样一来`R`就不够用了。

正确的答案是

```
"RRWWRRYYRR"
-> "R[Y]RWWRRYYRR"
-> "RYRW[W]WRRYYRR"
-> "RYRRRYYRR"
-> "RYYYRR"
-> "RRR"
-> ""
```

你会发现，有时候会为了**避免**过多的资源消除掉，向相邻的相同资源中插入不同的元素。

所以最终改进的剪枝策略：

- 相邻颜色相同，任意颜色都可以插入
- 相邻颜色不同，只能插入某个相同的颜色
- 对于首尾，只会插入相同的颜色



本题当然也可以使用BFS，但是并没有非常大的提升，主要的提升还是剪枝策略。

本题可以对`hand`进行压缩，用一个五位数表示，有细微的性能提升，提升还不如BFS。

# Code

```cpp
/*
 * @lc app=leetcode.cn id=488 lang=cpp
 *
 * [488] 祖玛游戏
 */

// @lc code=start
#include<bits/stdc++.h>
using namespace std;

template<typename T>
void show(vector<T> v) {
    for (auto x : v) cout << x << " ";
    cout << endl;
}


constexpr int MAX_INT = 1e9;
const array<char, 5> colors = {'R', 'Y', 'B', 'G', 'W'};

class Solution {
public:
    void eliminate (string & s) {
        for (int i = 0, j; i < s.length(); i = j) {
            for (j = i+1; j < s.length() && s[j] == s[i]; ++j);
            if (j - i >= 3) {
                s.erase(i, j - i);
                eliminate(s);
            }
        }
    }

    int dfs(const string && board, 
            const int hand, 
            map<pair<string, int>, int> & memo) {
        
        if (memo.find({board, hand}) != memo.end()) {
            return memo[{board, hand}];
        }
        if (board.empty()) return 0;

        int ret = MAX_INT;
        for (int i = 0, c = 1; i < 5; ++i, c *= 10)
            if (hand / c % 10 > 0) {
                // j = 0
                if (board[0] == colors[i]) {
                    string tmp = board;
                    tmp.insert(0, 1, colors[i]);
                    eliminate(tmp);
                    int r = dfs(move(tmp), hand - c, memo);
                    if (r != -1) ret = min(ret, r + 1);
                }
                int n = board.length();
                // j = n;
                if (board[n-1] == colors[i]) {
                    string tmp = board;
                    tmp.insert(n, 1, colors[i]);
                    eliminate(tmp);
                    int r = dfs(move(tmp), hand - c, memo);
                    if (r != -1) ret = min(ret, r + 1);
                }
                // j = 1..n-1
                for (int j = 1; j < board.size(); j++) {
                    if (board[j-1] == board[j]) {
                        string tmp = board;
                        tmp.insert(j, 1, colors[i]);
                        eliminate(tmp);
                        int r = dfs(move(tmp), hand - c, memo);
                        if (r != -1) ret = min(ret, r + 1);
                    }
                    else if (board[j-1] == colors[i] 
                        || board[j] == colors[i]) {
                        string tmp = board;
                        tmp.insert(j, 1, colors[i]);
                        eliminate(tmp);
                        int r = dfs(move(tmp), hand - c, memo);
                        if (r != -1) ret = min(ret, r + 1);
                    }
                }
            }
        memo[{board, hand}] = ret < MAX_INT ? ret : -1;
        return ret < MAX_INT ? ret : -1;
    }

    int findMinStep_DFS(string board, string hand) {
        map<pair<string, int>, int> memo;
        int _hand = 0;
        for (char ch : hand) {
            for (int i = 0, c = 1; i < 5; ++i, c *= 10) {
                if (colors[i] == ch)
                    _hand += c;
            }
        }
        return dfs(move(board), _hand, memo);
    }

    int findMinStep(string board, string hand) {
        int _hand = 0;
        for (char ch : hand) {
            for (int i = 0, c = 1; i < 5; ++i, c *= 10) {
                if (colors[i] == ch)
                    _hand += c;
            }
        }
        auto bfs = [this](string sb, int sh) {
            queue<tuple<string, int, int>> que;
            set<pair<string, int>> vis;

            que.push({sb, sh, 0});
            vis.insert({sb, sh});
            while (!que.empty()) {
                auto [b, h, d] = que.front();
                que.pop();
                if (b.empty()) return d;
                for (int i = 0, c = 1; i < 5; ++i, c *= 10) {
                    if (h / c % 10 == 0) continue;
                    int n = b.length();
                    // insert at 0
                    if (b[0] == colors[i]) {
                        string tmp = b;
                        tmp.insert(0, 1, colors[i]);
                        eliminate(tmp);
                        int _h = h - c;
                        if (vis.find({tmp, _h}) == vis.end()) {
                            vis.insert({tmp, _h});
                            que.push({move(tmp), move(_h), d+1});
                        }
                    }
                    // insert at n
                    if (b[n-1] == colors[i]) {
                        string tmp = b;
                        tmp.insert(n, 1, colors[i]);
                        eliminate(tmp);
                        int _h = h - c;
                        if (vis.find({tmp, _h}) == vis.end()) {
                            vis.insert({tmp, _h});
                            que.push({move(tmp), move(_h), d+1});
                        }
                    }
                    // insert at 1..n-1
                    for (int j = 1; j < n; ++j) {
                        if (b[j-1] == b[j]) {
                            string tmp = b;
                            tmp.insert(j, 1, colors[i]);
                            eliminate(tmp);
                            int _h = h - c;
                            if (vis.find({tmp, _h}) == vis.end()) {
                                vis.insert({tmp, _h});
                                que.push({move(tmp), move(_h), d+1});
                            }
                        }
                        else if (b[j-1] == colors[i]
                            || b[j] == colors[i]) {
                            string tmp = b;
                            tmp.insert(j, 1, colors[i]);
                            eliminate(tmp);
                            int _h = h - c;
                            if (vis.find({tmp, _h}) == vis.end()) {
                                vis.insert({tmp, _h});
                                que.push({move(tmp), move(_h), d+1});
                            }
                        }
                    }
                }
            }
            return -1;
        };
        return bfs(board, _hand);
    }
};
```

