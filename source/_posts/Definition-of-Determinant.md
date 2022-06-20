---
title: Definition of Determinant
tags: [determinant, matrix]
mathjax: true
date: 2022-06-17 04:42:32
categories:
- Math
- Matrix
---

本文讨论determinant（行列式）的定义。

<!--more-->

参考资料：

- [行列式的逆序数定义是怎么想出来的？ - 踧远陵歊的回答 - 知乎](https://www.zhihu.com/question/52957345/answer/589376055)
- [Laplace expansion - wiki](https://en.wikipedia.org/wiki/Laplace_expansion)


# 行列式定义

下面给出几种行列式的定义。

## 行列式的第一公理化构造

满足以下三条性质的函数 $\mathcal{D}:M_n(F) \rightarrow F$

1. （斜对称性）$\mathcal{D}(\ldots, a_i, \ldots, a_j , \ldots) = -\mathcal{D}(\ldots, a_j, \ldots, a_i, \ldots)$
2. （多重线性）$\mathcal{D}(\ldots, c_1a_1+c_2a_2\cdots,\ldots) = c_1\mathcal{D}(\ldots, a_1, \ldots) + c_2\mathcal{D}(\ldots, a_2, \ldots) + \cdots$
3. （正规化条件）$\mathcal{D}(E) = 1$

是唯一确定的，称作矩阵$A$的行列式。数域 $F$ 可以是 $\mathbb{R}$ 或者 $\mathbb{C}$ 。


## 第二公理化构造

函数 $\mathcal{D}$ 需要满足以下三个条件：

1. $\mathcal{D}(\ldots, \lambda a_i, \ldots) = \lambda \mathcal{D}(\ldots, a_i, \ldots)$
2. $\mathcal{D}(\ldots, a_i + a_j, \ldots , a_j, \ldots) = \mathcal{D}(\ldots, a_i, \ldots, a_j, \ldots)$
3. $\mathcal{D}(E) = 1$

## 余子式展开的归纳定义

$$\mathcal{D}(A) = \sum_{j=1}^n (-1)^{i+j} a_{ij} M_{ij}$$

## 通过乘法性质的刻画

函数 $\mathcal{D}$ 需要满足下面三个条件：

1. $\forall A, B \in M_n, \mathcal{D}(AB) = \mathcal{D}(A)\mathcal{D}(B)$
2. 对于 $E(i,j)$, $\mathcal{D}(E(i,j)) = -1$，$E(i,j)$ 表示交换两行的初等变换。
3. 对于主对角线为 $(\lambda, 1, 1, \ldots, 1)$ 的上三角矩阵 $A$，$\mathcal{D}(A) = \lambda$

# 行列式几种定义的等价性

## 第一公理推出第二公理

$$
\begin{align}
& \because \mathcal{D}\left(\ldots, \sum_{i} c_i a_i , \ldots\right) = \sum_{i}c_i\mathcal{D}\left(\ldots, a_i, \ldots \right) \\
& \therefore \mathcal{D}\left( \ldots, \lambda a, \ldots \right) = \lambda \mathcal{D}\left( \ldots, a, \ldots \right) \\
& \because \mathcal{D}\left(\ldots, a, \ldots, a, \ldots\right) = - \mathcal{D}\left(\ldots, a,\ldots, a, \ldots \right) \\
& \therefore \mathcal{D}\left(\ldots, a, \ldots, a, \ldots \right)=0 \\
& \therefore \mathcal{D}\left(\ldots, a_i+a_j, \ldots, a_j, \ldots\right) \\
 &= \mathcal{D}\left(\ldots, a_i, \ldots, a_j, \ldots \right) + \mathcal{D}\left(\ldots, a_j, \ldots, a_j, \ldots \right) \\
 & = \mathcal{D}\left(\ldots, a_i, \ldots, a_j, \ldots \right) \\
\end{align}
$$

## 第二公理推出第一公理

todo!

## Laplace expansion

设 $A \in M_n$，$i,j \in \lbrace 1,2,\ldots,n\rbrace$，$B$ 的元素为 $b_{i,j}$，$M_{i,j}$ 为 $B$ 去除第 $i$ 行 $j$ 列的余子式，$a_{s,t}$ 为 $M_{i,j}$ 的元素。

考虑 $\det B$ 中包含 $b_{i,j}$ 的项，具有以下形式：

$$\begin{align}
& \text{sgn}(\tau) b_{1,\tau(1)} b_{2, \tau(2)} \cdots b_{i,j} \cdots b_{n, \tau(n)} \\ 
= & \text{sgn}(\tau) b_{i,j} b_{1,\tau(1)} b_{2,\tau(2)} \cdots b_{i-1,\tau(i-1)} b_{i+1,\tau(i+1) \cdots b_{n, \tau(n)}} \\
= & \text{sgn}(\tau) b_{i,j} a_{1, \sigma(1)} a_{2, \sigma(2)} \cdots a_{i-1, \sigma(i-1)} a_{i, \sigma(i)} \cdots a_{n-1, \sigma(n-1)}
\end{align}$$

其中 $\tau \in S_n$ 是某个排列，$j = \tau(i)$ ， 存在一个唯一的排列 $\sigma \in S_{n-1}$，事实上 $\sigma \leftrightarrow \tau$ 构成了 $S_{n-1}$ 和 $\lbrace \tau \in S_n : \tau(i)=j \rbrace$ 之间的一一映射。$\tau$ 和 $\sigma$ 之间的关系如下：

$$\sigma = 
\begin{pmatrix}  
1 & 2 & \cdots & i-1 & i & \cdots & n-1 \\ 
(\leftarrow)_j \tau(1) & (\leftarrow)_j \tau(2) & \cdots & (\leftarrow)_j \tau(i-1) & (\leftarrow)_j \tau(i+1)&  \cdots & (\leftarrow)_j \tau(n)
\end{pmatrix}$$

其中 $(\leftarrow)_j = (n, n-1, \ldots, j+1, j)$ 表示一个cycle，其含义是将大于 $j$ 的数字减去 $1$，从而将所有数字映射到 $\lbrace 1, 2, \ldots, n-1 \rbrace$ 。

排列 $\tau$ 也可以由 $\sigma$ 得到。首先定义 $\sigma'\in S_n$，对于 $1\leq k \leq n-1$，$\sigma'(k) = \sigma(k)$，并且 $\sigma'(n) = n$，即：

$$\sigma' = 
\begin{pmatrix}  
1 & 2 & \cdots & i-1 & i & \cdots & n-1 & n\\ 
(\leftarrow)_j \tau(1) & (\leftarrow)_j \tau(2) & \cdots & (\leftarrow)_j \tau(i-1) & (\leftarrow)_j \tau(i+1)&  \cdots & (\leftarrow)_j \tau(n) & n
\end{pmatrix}$$

首先作用 $(\leftarrow)_i$ 然后作用 $\sigma'$ 得到：

$$\sigma' (\leftarrow)_i = 
\begin{pmatrix}  
1 & 2 & \cdots & i-1 & i+1 & \cdots & n & i\\ 
(\leftarrow)_j \tau(1) & (\leftarrow)_j \tau(2) & \cdots & (\leftarrow)_j \tau(i-1) & (\leftarrow)_j \tau(i+1)&  \cdots & (\leftarrow)_j \tau(n) & n
\end{pmatrix}$$

首先作用 $\tau$ 然后作用 $(\leftarrow)_j$ 得到：

$$(\leftarrow)_j \tau = 
\begin{pmatrix}  
1 & 2 & \cdots & i-1 & i & i+1 & \cdots & n\\ 
(\leftarrow)_j \tau(1) & (\leftarrow)_j \tau(2) & \cdots & (\leftarrow)_j \tau(i-1) & n & (\leftarrow)_j \tau(i+1) & \cdots & (\leftarrow)_j \tau(n)
\end{pmatrix}$$

观察可得 $\sigma' (\leftarrow)_i = (\leftarrow)_j \tau$，所以

$$\begin{align}
\tau & = (\rightarrow)_j \sigma' (\leftarrow)_i \\
 & = \begin{pmatrix}j & j+1 & \cdots & n \end{pmatrix} \sigma' \begin{pmatrix} n & n-1 & \cdots &  i \end{pmatrix} \\
\text{sgn}(\tau) & = \text{sgn}(\sigma') (-1)^{n-j + n-i} = \text{sgn}(\sigma) (-1)^{i+j}
\end{align}$$

{% note info %}
长度为 $n$ 的 cycle $c_n$ 可以分解为 $n-1$ 次交换，所以 $\text{sgn}(c_n) = (-1)^{n-1}$ 。
{% endnote %}


既然 $\tau \leftrightarrow \sigma$ 是一一映射，所以

$$\begin{align}
\sum_{i=1}^n \sum_{\tau \in S_n : \tau(i) = j} \text{sgn}(\tau) b_{1, \tau(1)} \cdots b_{n, \tau(n)}
& = \sum_{i=1}^n \sum_{\sigma \in S_{n-1}} (-1)^{i+j} \text{sgn}(\sigma) b_{i,j} a_{1,\sigma(1)} \cdots a_{n-1, \sigma(n-1)} \\
& = \sum_{i=1}^n b_{i,j} (-1)^{i+1} \sum_{\sigma \in S_{n-1}} \text{sign}(\sigma) a_{1,\sigma(1)} \cdots a_{n-1, \sigma(n-1)} \\
& = \sum_{i=1}^n b_{i,j} (-1)^{i+j} M_{i,j}
\end{align}$$


# 由公理生成定义

$\forall A \in M_n, A = (a_1, \ldots, a_n)$，其中 $a_i$ 表示第 $i$ 行， $E=(e_1, \ldots, e_n)$ 表示单位矩阵。

$$\begin{align} \mathcal{D}(A) & = \mathcal{D}(a_1, \ldots, a_n) \\
& = \mathcal{D}\left(\sum_{i=1}^n a_{1i}e_i, \ldots, a_n \right) \\
& = \sum_{i=1}^n a_{1i} \mathcal{D}\left(e_i, \ldots, a_n \right)\\
& = \sum_{i=1}^n a_{1i} \mathcal{D} \left( e_i, \sum_{j=1}^n a_{2j}e_j\ldots, a_n \right) \\ 
& = \sum_{i=1}^n \sum_{j=1}^n a_{1i} a_{2j} \mathcal{D}\left( e_i, e_j, \ldots, a_n \right) \\
& = \sum_{i=1}^n \sum_{j=1}^n \cdots \sum_{k=1}^n a_{1i} a_{2j} \cdots a_{nk} \mathcal{D}\left(e_i, e_j, \ldots, e_k \right) \end{align}$$

根据{% label @第一公理 %}，可以证明**引理**：$\mathcal{D}(\ldots, a, \ldots, a, \ldots) = 0$。

$$\begin{align}
\mathcal{D}\left(A \right) & = \sum_{i_1} \sum_{i_2 \neq i_1} \cdots \sum_{i_n \neq i_1 \land i_n \neq i_2 \land \cdots} a_{1i_1} a_{2i_2} \cdots a_{ni_n} \mathcal{D}\left(e_{i_1}, \ldots, e_{i_n} \right) \\ 
& = \sum_{\sigma\in S_n} \mathcal{D}\left(e_{\sigma(1)}, \ldots, e_{\sigma(n)} \right) \prod_{i=1}^n a_{i \sigma(i)}
\end{align}$$

下面就需要确定 $\mathcal{D}\left(e_{\sigma(1)}, \ldots, e_{\sigma(n)} \right)$ 的符号，通过交换变成 $\mathcal{D}\left(e_1, \ldots, e_n \right)$ 。设 $h = \sigma^{-1}$ 。

- $e_1$ 所在的位置是 $h(1)$，需要移动到 $1$ ，通过交换 $k_1$ 次；
- $e_2$ 移动到 $2$ 需要交换 $k_2$ 次；

不难发现，$k_1$ 等于比 $1$ 大但是在它前面的数字个数，也就是逆序对数，$k_2$ 等于比 $2$ 大但是在其前面的数字个数，也是逆序对数，所以 $k_1+k_2+\cdots + k_n = k$ 等于总的逆序对数目，如果 $k \equiv 0 (\text{mod } 2)$，则称这个排列 $\sigma$ 是{% label @偶排列 %}，否则为{% label @奇排列 %}。

$$\mathcal{D}\left(A \right) = \sum_{\sigma \in S_n} \varepsilon_\sigma \prod_{i=1}^n a_{i \sigma(i)}$$

通过最终的定义式，容易看出，总共有 $n!$ 项求和，因为 $S_n$ 全排列的数目是 $n!$ ，每项由 $n$ 项因子组成，这 $n$ 项因子来自矩阵 $A$ 的不同行不同列，不难看出行和列的地位是相等的。

