---
title: Leetcode 753
categories:
  - 刷题
  - Leetcode
tags: [欧拉回路]
date: 2022-03-06 04:51:12
mathjax: true
---

[Leetcode 753](https://leetcode-cn.com/problems/cracking-the-safe/)

<!--more-->

# Description

有一个需要密码才能打开的保险箱。密码是 $n$ 位数, 密码的每一位是 $k$ 位序列 $0, 1, \ldots, k-1 $ 中的一个 。

你可以随意输入密码，保险箱会自动记住最后 $n$ 位输入，如果匹配，则能够打开保险箱。

举个例子，假设密码是 "345"，你可以输入 "012345" 来打开它，只是你输入了 6 个字符.

请返回一个能打开保险箱的最短字符串。

# Sample

示例1:

```
输入: n = 1, k = 2
输出: "01"
说明: "10"也可以打开保险箱。
```

示例2:

```
输入: n = 2, k = 2
输出: "00110"
说明: "01100", "10011", "11001" 也能打开保险箱。
```

# Hint

- $n$ 的范围是 $[1, 4]$。
- $k$ 的范围是 $[1, 10]$。
- $k^n$ 最大可能为 $4096$。


# Solution

## 转化为图

本题看上去没什么思路，像是一个编码题。

因为密码是 $n$ 位，所以可以看作一个大小为 $n$ 的框在移动，每次向后移动一位，这么看好像还是没有什么意义。

其实，如果把移动看作状态转移，把前 $n-1$ 位看作状态，整个移动的过程就是一个 DFA 的状态转移过程。这是本题的关键！DFA总共有 $k^{n-1}$ 个状态，表示密码的前 $n-1$ 位，每个状态都有 $k$ 条转移，转移的终点可以计算出来，显然对于每个结点都有 $d_{in}(v) = d_{out}(v) = k$ 。

本题的目标是什么，求一个最短的输入串，使得该 DFA 把所有的边遍历至少一次。显然如果该 DFA 存在欧拉回路就好了，欧拉回路就是我们的答案。

## 欧拉回路

- 定义：每条边遍历一次的回路
- 判定：连通图，所有结点的入度等于出度 $d_{in}(v) = d_{out}(v)$

该判定条件是充分必要的，必要性很简单，对于一个欧拉回路 $x_0, x_1,\ldots, x_n, x_0$ ，由于所有边都在路径中，所以每个结点的入度一定等于出度。

充分性，考虑这么一个过程，从一个结点 $u$ 出发，沿着边任意的移动，每次经过一条边就将其删除，直到走到结点 $u'$ 无法移动为止，路径为 $u, \ldots, v, \ldots, u'$ 。首先可以证明 $u'=u$，因为路径中间的结点 $v$ 都进入一次、出去一次，经过删除后一定仍然满足 $d_{in}(v) = d_{out}(v)$，如果 $u' \neq u$ 那么 $u'$ 一定还有边可以出去，这和无法移动矛盾，所以 我们找到了一个环 $C = u\ldots v\ldots u$ 。接下来，假设 $d(v) \neq 0$，我们从 $v$ 开始再重复上述过程找到环 $C'$，显然 $C'$ 可以嵌入到 $C$ 中构成一个更大的环，不断重复就找到了欧拉回路。

充分性的证明同时也提供了计算欧拉回路的算法，也证明了欧拉图一定可以拆分为若干环的并，并且这些环只会共享结点，不可能共享边。

## Hierholzer算法

代码极为简单

```cpp
void dfs(int x, vector<list<int>> & G, list<int> & trace) {
  while (!G[x].empty()) {
    int y = G[x].back();
    G[x].pop_back();
    dfs(y, G, trace);
  }
  trace.push_front(x);
}
```

算法的关键，当走无可走的时候，把结点压入 **栈** 中。

这里给一种不太严谨的证明：首先考虑一个环，正确性显然；在一个环上嵌入另一个环，不妨设嵌入点为 $v$ ，自己手动模拟一下，在回溯的过程中，当回溯到 $v$ 时，进入新的环；更多的环也是同样的道理，可以归纳法证明。

# Code

```cpp
class Solution {
private:
    int N, K;
public:
    void dfs(int x, vector<list<int>> & G, string & s) {
        while (!G[x].empty()) {
            int y = G[x].back();
            int e = y % K;
            G[x].pop_back();
            dfs(y, G, s);
            s.push_back('0' + e);
        }
        // trace.push_front(x);
    }
    
    string crackSafe(int n, int k) {
        if (n == 1) {
            string ans;
            for (int i = 0; i < k; ++i)
                ans.push_back('0' + i);
            return ans;
        }
        N = n, K = k;

        int P = pow(k, n-1);

        vector<list<int>> G(P);

        for (int i = 0; i < P; ++i) {
            for (int j = 0; j < k; ++j) {
                G[i].push_back((i * k + j) % P);
            }
        }

        string ans;
        dfs(0, G, ans);
        for (int i = 0; i < n-1; ++i)
            ans.push_back('0');
        return ans;
    }
};
```