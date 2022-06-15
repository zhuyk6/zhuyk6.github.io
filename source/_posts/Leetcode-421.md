---
title: Leetcode 421
categories:
  - 刷题
  - Leetcode
tags: [位运算, Trie]
date: 2021-07-04 23:10:07
mathjax:
---

[Leetcode 421](https://leetcode-cn.com/problems/maximum-xor-of-two-numbers-in-an-array/)

<!--more-->

# Description

给你一个整数数组 `nums` ，返回 `nums[i] XOR nums[j]` 的最大运算结果，其中 `0 ≤ i ≤ j < n` 。

进阶：你可以在 `O(n)` 的时间解决这个问题吗？


# Sample

## Sample 1

```
输入：nums = [3,10,5,25,2,8]
输出：28
解释：最大运算结果是 5 XOR 25 = 28.
```

## Sample 2

```
输入：nums = [0]
输出：0
```

## Sample 3：

```
输入：nums = [2,4]
输出：6
```

## Sample 4：

```
输入：nums = [8,10,2]
输出：10
```

## Sample 5：

```
输入：nums = [14,70,53,83,49,91,36,80,92,51,66,70]
输出：127
```

# Hint

- `1 <= nums.length <= 2 * 10^4`
- `0 <= nums[i] <= 2^31 - 1`

# Solution

非常nice的一道题目，暴力肯定不可取，平方的复杂度不能接受。因为题目要求的是异或运算的结果，所以肯定是从二进制出发来考虑问题。

非常trivial的一个想法，优先使高位的结果为1，`x xor y`等于1，需要`x`和`y`不相同，所以很自然的，我们将所有数字按照第`b`位分成两组，如果两组都不为空，那么显然`z`的第`b`位一定可以等于`1`，接下来继续考虑`b-1`位。

当然这么递归就把问题搞麻烦了，导致`x`和`y`都一直在变，我们可以固定一个`x=nums[i]`，然后在`nums[0]..nums[i-1]`里面挑选`y`使得结果最大。

所以我们需要维护一个数据结构，能够快速添加一个数字，并且根据二进制位查询数字，可以根据二进制建立索引树，从高位向低位，这也是一个前缀树Trie，是IntTrie的一种，HAMT的思想与之类似。

# Code

```cpp
struct Node {
    Node *lc, *rc;

    Node () : lc(nullptr), rc(nullptr) {}
};

class IntTrie {
private:
    Node * root;
public:
    IntTrie () {
        root = new Node();
    }

    void insert(int num) {
        Node * t = root;
        for (int i = 30; i >= 0; --i) {
            if ((num & (1 << i)) == 0) {
                if (t->lc == nullptr)
                    t->lc = new Node();
                t = t->lc;
            }
            else {
                if (t->rc == nullptr)
                    t->rc = new Node();
                t = t->rc;
            }
        }

    }

    int ask(int num) const {
        Node * t = root;
        int ret = 0;
        for (int i = 30; i >= 0; --i) {
            if (num & (1 << i)) {
                if (t->lc != nullptr) {
                    ret |= (1 << i);
                    t = t->lc;
                }
                else
                    t = t->rc;
            }
            else {
                if (t->rc != nullptr) {
                    ret |= (1 << i);
                    t = t->rc;
                }
                else
                    t = t->lc;
            }
        }
        return ret;
    }
};

class Solution {
public:
    int findMaximumXOR(vector<int>& nums) {
        IntTrie t;
        int ret = 0;
        for (int v : nums) {
            t.insert(v);
            ret = max(ret, t.ask(v));
        } 
        return ret;
    }
};
```