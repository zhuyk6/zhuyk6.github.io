---
title: Leetcode 1896
categories:
  - 刷题
  - Leetcode
tags: [栈, 算术表达式]
date: 2021-06-29 17:06:52
mathjax:
---

[Leetcode 1896](https://leetcode-cn.com/problems/minimum-cost-to-change-the-final-value-of-expression/)

<!--more-->

# Description

给你一个 **有效的** 布尔表达式，用字符串 `expression` 表示。这个字符串包含字符 `'1'`，`'0'`，`'&'`（按位 **与** 运算），`'|'`（按位 **或** 运算），`'('` 和 `')'` 。

- 比方说，`"()1|1"` 和 `"(1)&()"` **不是有效** 布尔表达式。而 `"1"`， `"(((1))|(0))"` 和 `"1|(0&(1))"` 是 **有效** 布尔表达式。

你的目标是将布尔表达式的 **值** 反转 （也就是将 `0` 变为 `1` ，或者将 `1` 变为 `0`），请你返回达成目标需要的 **最少操作** 次数。

- 比方说，如果表达式 `expression = "1|1|(0&0)&1"` ，它的 **值** 为 `1|1|(0&0)&1 = 1|1|0&1 = 1|0&1 = 1&1 = 1` 。我们想要执行操作将 **新的** 表达式的值变成 `0` 。

可执行的 **操作** 如下：

- 将一个 `'1'` 变成一个 `'0'` 。
- 将一个 `'0'` 变成一个 `'1'` 。
- 将一个 `'&'` 变成一个 `'|'` 。
- 将一个 `'|'` 变成一个 `'&'` 。

**注意**：`'&'` 的 **运算优先级** 与 `'|'` 相同 。计算表达式时，括号优先级 **最高** ，然后按照 **从左到右** 的顺序运算。


# Sample

## Sample 1

```
输入：expression = "1&(0|1)"
输出：1
解释：我们可以将 "1&(0|1)" 变成 "1&(0&1)" ，执行的操作为将一个 '|' 变成一个 '&' ，执行了 1 次操作。
新表达式的值为 0 。
```

## Sample 2

```
输入：expression = "(0&0)&(0&0&0)"
输出：3
解释：我们可以将 "(0&0)&(0&0&0)" 变成 "(0|1)|(0&0&0)" ，执行了 3 次操作。
新表达式的值为 1 。
```

## Sample 3

```
输入：expression = "(0|(1|0&1))"
输出：1
解释：我们可以将 "(0|(1|0&1))" 变成 "(0|(0|0&1))" ，执行了 1 次操作。
新表达式的值为 0 。
```

# Hint

- `1 <= expression.length <= 10^5`
- `expression` 只包含 `'1'`，`'0'`，`'&'`，`'|'`，`'('` 和 `')'`
- 所有括号都有与之匹配的对应括号。
- 不会有空的括号（也就是说 `"()"` 不是 `expression` 的子字符串）。

# Solution

很有意思的一道题目，乍一看貌似很复杂。

很直观的一个想法，先把**表达式**`expr`的值求出来，然后计算`make expr (not $ eval expr)`。`make`函数的构造十分简单，如下：

```haskell Haskell version for "make"
data Expr = Bool Bool | (&) Expr Expr | (|) Expr Expr

make :: Expr -> Bool -> Int
make (Bool True) True = 0
make (Bool True) False = 1
make (Bool False) True = 1
make (Bool False) False = 0
make (a & b) True = min (make a True + make b True) (1 + min (make a True) (make b True))
make (a & b) False = min (make a False) (make b False)
make (a | b) True = min (make a True) (make b True)
make (a | b) False = min (make a False + make b False) (1 + min (make a False) (make b False))
```

所以只需要把`s`给Parse成`Expr`，问题就解决了。而对于算术表达式，我们可以使用栈在线性时间内计算。

# Code

```cpp
class Solution {
public:
    int minOperationsToFlip(string expression) {
        auto make = [](bool x, int n, bool target) {
            return x == target ? 0 : n;
        };

        auto merge = [&](bool x, int n1, bool y, int n2, char op) {
            bool z;
            int n3;
            if (op == '&') {
                z = x && y;
                if (z) { // x && y -> false
                    n3 = min(make(x, n1, false), make(y, n2, false));
                }
                else {  // x && y -> true
                    n3 = min(make(x, n1, true) + make(y, n2, true), 
                        1 + min(make(x, n1, true), make(y, n2, true)));
                }
            }
            else {
                z = x || y;
                if (z) { // x || y -> false
                    n3 = min(make(x, n1, false) + make(y, n2, false), 
                        1 + min(make(x, n1, false), make(y, n2, false)));
                }
                else {  // x || y -> true
                    n3 = min(make(x, n1, true), make(y, n2, true));
                }
            }
            return make_pair(z, n3);
        };

        stack<char> ops;
        stack<pair<bool, int>> exprs;
        expression = "(" + expression + ")";
        for (auto c: expression) {
            if (c == '(') {
                ops.push(c);
            }
            else if (c == ')') {
                while (ops.top() != '(') {
                    auto [y, n2] = exprs.top();
                    exprs.pop();
                    auto [x, n1] = exprs.top();
                    exprs.pop();
                    char op = ops.top();
                    ops.pop();
                    auto [z, n3] = merge(x, n1, y, n2, op);
                    exprs.push(make_pair(z, n3));
                }
                ops.pop();
            }
            else if (c == '&' || c == '|') {
                while (ops.top() != '(') {
                    auto [y, n2] = exprs.top();
                    exprs.pop();
                    auto [x, n1] = exprs.top();
                    exprs.pop();
                    char op = ops.top();
                    ops.pop();
                    auto [z, n3] = merge(x, n1, y, n2, op);
                    exprs.push(make_pair(z, n3));
                }
                ops.push(c);
            }
            else if (c == '0'){
                exprs.push(make_pair(false, 1));
            }
            else if (c == '1') {
                exprs.push(make_pair(true, 1));
            }
            else {
                printf("Parse Error!\n");
            }
        }
        return exprs.top().second;
    }
};
```