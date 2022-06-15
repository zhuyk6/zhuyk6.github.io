---
title: Leetcode 1851
tags: [数据结构, map, heap]
date: 2021-06-24 16:52:44
categories:
- 刷题
- Leetcode
mathjax: true
---

[Leetcode 1851](https://leetcode-cn.com/problems/minimum-interval-to-include-each-query/)

<!--more-->

# Description

给你一个二维整数数组 `intervals` ，其中 `intervals[i] = [lefti, righti]` 表示第 `i` 个区间开始于 `lefti` 、结束于 `righti`（包含两侧取值，闭区间）。区间的 长度 定义为区间中包含的整数数目，更正式地表达是 `righti - lefti + 1` 。

再给你一个整数数组 `queries` 。第 `j` 个查询的答案是满足 `lefti <= queries[j] <= righti` 的 长度最小区间 `i` 的长度 。如果不存在这样的区间，那么答案是 `-1` 。

以数组形式返回对应查询的所有答案。

# Sample

## Sample 1

```
输入：intervals = [[1,4],[2,4],[3,6],[4,4]], queries = [2,3,4,5]
输出：[3,3,1,4]
解释：查询处理如下：

- Query = 2 ：区间 [2,4] 是包含 2 的最小区间，答案为 4 - 2 + 1 = 3 。
- Query = 3 ：区间 [2,4] 是包含 3 的最小区间，答案为 4 - 2 + 1 = 3 。
- Query = 4 ：区间 [4,4] 是包含 4 的最小区间，答案为 4 - 4 + 1 = 1 。
- Query = 5 ：区间 [3,6] 是包含 5 的最小区间，答案为 6 - 3 + 1 = 4 。
```

## Sample 2

```
输入：intervals = [[2,3],[2,5],[1,8],[20,25]], queries = [2,19,5,22]
输出：[2,-1,4,6]
解释：查询处理如下：

- Query = 2 ：区间 [2,3] 是包含 2 的最小区间，答案为 3 - 2 + 1 = 2 。
- Query = 19：不存在包含 19 的区间，答案为 -1 。
- Query = 5 ：区间 [2,5] 是包含 5 的最小区间，答案为 5 - 2 + 1 = 4 。
- Query = 22：区间 [20,25] 是包含 22 的最小区间，答案为 25 - 20 + 1 = 6 。
```


# Hint

$1 \leq intervals.length \leq 10^5$

$1 \leq queries.length \leq 10^5$

$queries[i].length = 2$

$1 \leq left_i \leq right_i \leq 10^7$

$1 \leq queries[j] \leq 107$

# Solution

非常典的区间覆盖+点询问。

$$ans_i = \min \lbrace R_j - L_j + 1 : L_j \leq Q_i \leq R_j \rbrace$$

思路有很多，在线离线都可以，关键在于选取合适的数据结构。

## 在线+线段树

我第一时间想的是在线算法，使用线段树维护，在区间 $[L_i, R_i]$ 插入 $R_i-L_i+1$， 维护最小值，点询问。整个复杂度是 $O(Q\log M)$，其中 $Q$ 是询问总数，$M$ 是区间大小。问题是本题 $M = 10^7$，太大了，耗费的空间 $O(M)$ 太多，不是好的选择（可以使用函数式线段树来优化空间到 $O(N)$）。

## 离线+堆（平衡树）

本题可以使用离线算法，那不用白不用，可以提前处理，减少一些偏序关系组成的约束。

我的想法：首先将`intervals`按照`(L, R)`来排序，将`Q`也排好序，这样对于逐渐增大的 $q\in Q$，跟着不断将线段添加进数据结构即可。问题在于，选取什么数据结构，用来维护什么？

我的思路是：因为 $L\leq Q$ 的约束已经干掉了，只需要考虑 $Q\leq R$ 的约束，但是 $R$ 是无序的，所以用数据结构来维护 $R$ 使其有序，询问 $R_i \geq Q$ 的里面长度最小的。

有序的数据结构很多，最容易想到的是平衡树，根据 $R$ 建立平衡树，每次询问 $Q$ 可以快速定位到那些大于 $Q$ 的子树，在平衡树上维护一个域来存储线段长度和`min`，`min`表示大于等于本节点的最小线段长度，可以在维护平衡树的同时 $O(1)$ 的维护`min`。

其实更进一步，因为 $Q$ 已经排好序的，所以对于那些 $R < Q$ 的线段就在也用不到了，直接删除即可。所以我们不需要平衡树，平衡树的左边永远用不到。直接对 $R$ 维护一个小根堆，当堆顶小于 $Q$ ，就`pop`。同样我们要维护线段长度和`min`，这个就更简单了，`min`就是节点和儿子中最小的。

无论是用平衡树还是堆，性能都十分优秀，唯一恶心的问题是必须手写，因为这个辅助域的改造。

## 离线+`map`

标准答案更进一步，我们为啥要维护 $R$ 呢，有用的数据不是线段长度吗？

其实可以更进一步，不把线段当作整体排序，直接把 $L_i, R_i, Q_j$ 拆开放在一起排序，毕竟约束就是这三者的偏序。

排好序从左往右扫，

- 遇到 $L_i$ ，将 $(R_i - L_i + 1)$ 添加进数据结构；
- 遇到 $R_i$ ，将 $(R_i - L_i + 1)$ 从数据结构中删除；
- 遇到 $Q_j$ ，询问数据结构中的最小元素。

那么很简单了，数据结构要允许添加重复元素和删除某个元素，并查询最小值，平衡树就完事了。

{% note warning %}
	`CPP`无比之坑，`multiset`可以添加重复元素，但是`erase`会将所有该元素删除。所以直接使用`map`省事，有了`map`，这`multiset`有屁用阿。
{% endnote %}


# Code

## Heap

```cpp
class Heap {
private:
    int tot;
    vector<pair<pair<int, int>, int> > a;

    void update(int x) {
        a[x].second = a[x].first.second;
        if ((x<<1) <= tot) {
            a[x].second = min(a[x].second, a[x<<1].second);
        }
        if ((x<<1|1) <= tot) {
            a[x].second = min(a[x].second, a[x<<1|1].second);
        }
    }

    void push_up(int x) {
        while (x > 1) {
            if (a[x].first < a[x>>1].first) {
                swap(a[x], a[x>>1]);
            }
            update(x);
            update(x>>1);
            x = x>>1;
        }
    }

    void push_down(int x) {
        if ((x<<1) <= tot && (x<<1|1) <= tot) {
            int y = a[x<<1].first < a[x<<1|1].first ? (x<<1) : (x<<1|1);
            if (a[x].first > a[y].first) {
                swap(a[x], a[y]);
                update(x);
                update(y);
                push_down(y);
            }
        }
        else if ((x<<1) <= tot) {
            int y = x<<1;
            if (a[x].first > a[y].first) {
                swap(a[x], a[y]);
                update(x);
                update(y);
                push_down(y);
            }
        }
        update(x);
        if (x > 1) update(x >> 1);
    }

public:
    Heap(){
        tot = 0;
        a.push_back(make_pair(make_pair(-1, -1), -1));
    }

    void push(pair<int, int> v) {
        ++tot;
        a.push_back(make_pair(v, v.second));
        push_up(a.size()-1);
    }

    void pop() {
        swap(a[1], a[tot]);
        update(1);
        a.pop_back();
        --tot;
        push_down(1);
    }

    int size() const {
        return tot;
    }

    pair<pair<int, int>, int> top() const {
        return a[1];
    }
};

class Solution {
public:
vector<int> minInterval_old(vector<vector<int>>& intervals, vector<int>& queries) {
        auto cmp1 = [](const vector<int> & a, const vector<int> & b) {
            return a[0] < b[0] || a[0] == b[0] && a[1] < b[1];
        };
        sort(intervals.begin(), intervals.end(), cmp1);

        // for (auto v: intervals) {
        //     cout << "(" << v[0] << ", " << v[1] << ")\n";
        // }

        int m = intervals.size();
        int n = queries.size();

        vector<int> q_sorted(n);
        vector<int> q_answer(n);
        for (int i = 0; i < n; ++i) q_sorted[i] = i;
        auto cmp2 = [&](int x, int y) {
            return queries[x] < queries[y];
        };
        sort(q_sorted.begin(), q_sorted.end(), cmp2);

        // for (auto id: q_sorted) cout << id << " "; cout << endl;

        Heap h;
        int i = 0;
        for (int j = 0; j < n; ++j) {
            while (i < m && intervals[i][0] <= queries[q_sorted[j]]) {
                h.push(make_pair(
                    intervals[i][1], 
                    intervals[i][1] - intervals[i][0] + 1
                ));
                ++i;
            }
            while (h.size() > 0 && 
                    h.top().first.first < queries[q_sorted[j]]) {
                h.pop();
            }
            // cout << "ask " << queries[q_sorted[j]] << endl;
            // cout << "heap size = " << h.size() << endl;
            if (h.size() > 0) {
                // cout << h.top().first.first << " - " << h.top().first.second << endl;
                q_answer[q_sorted[j]] = h.top().second;
            }
            else {
                q_answer[q_sorted[j]] = -1;
            }
        }

        return q_answer;
    }
};
```

## map

```cpp
class Solution{
public:
	vector<int> minInterval(vector<vector<int>>& intervals, vector<int>& queries) {
        int m = intervals.size();
        int n = queries.size();
        vector<pair<int, pair<int, int> > > a;
        for (int i = 0; i < m; ++i) {
            a.push_back(make_pair(intervals[i][0], make_pair(1, i)));
            a.push_back(make_pair(intervals[i][1], make_pair(3, i)));
        }
        for (int i = 0; i < n; ++i) {
            a.push_back(make_pair(queries[i], make_pair(2, i)));
        }
        sort(a.begin(), a.end());
        map<int, int> s;
        vector<int> answer(n);
        for (auto [x, p] : a) {
            int t = p.first, i = p.second;

            // printf("x=%d, t=%d, i=%d\n", x, t, i);

            if (t == 1) {
                s[intervals[i][1] - intervals[i][0] + 1] += 1;
            }
            else if (t == 3) {
                int v = intervals[i][1] - intervals[i][0] + 1;
                s[v] -= 1;
                if (s[v] == 0) s.erase(v);
            }
            else {
                answer[i] = s.empty() ? -1 : s.begin()->first;
            }
            // printf("set = ");
            // for (auto v: s) printf("%d ", v); printf("\n");
        }
        return answer;
    }
};
```