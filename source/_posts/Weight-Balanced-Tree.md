---
title: Weight Balanced Tree
tags: [Balanced tree]
mathjax: true
date: 2021-08-21 11:18:14
categories:
- Data Structures
---

本文介绍Weight-Balanced tree（WB tree）。

参考资料：

- 2010论文：[Balancing weight-balanced trees](https://yoichihirai.com/bst.pdf?utm_source=pocket_mylist)
- 1972论文
- 1992论文：Implementing Sets Efficiently in a Functional Language, Stephen Adams

<!--more-->



# Overview

平衡树有很多种，对于“平衡”条件的选择也有很多。AVL和RB-tree是对height树高进行控制，自然也可以对weight进行控制。

weight是什么，是一个和size有关的函数，在最早的1972年提出WB tree的论文（下面我们称之为原始WB tree）中，weight被定义为 $size+1$。在1992年的论文中，则直接使用 size 作为weight。

WB-tree的思想十分简单，我们不希望左右子树差距过大，所以限制 $1/\Delta \leq w_l / w_r \leq \Delta$ 。如果经过插入、删除操作后，导致不再平衡，就要通过旋转来重新平衡。事实上限制子树和自身的比值也可以，都是可以互相转化的。

不妨设 $w_r > \Delta \cdot w_l$，此时右子树过大，需要将右子树的部分转移到左边，就有两种情况：

1. $w_b < \alpha \cdot w_c$：此时进行单旋；
{% mermaid graph TD %}

subgraph single rotate
x2((x)) --> a2[a]
x2 --> b2[b]
y2((y)) --> x2
y2 --> c2[c]
end

subgraph case1
x1((x)) --> a1[a]
x1 --> y1((y))
y1 --> b1[b]
y1 --> c1[c]
end
{% endmermaid %}

2. $w_b \geq \alpha \cdot w_c$：此时进行双旋。
{% mermaid graph TD %}

subgraph double rotate
b2((b)) --> x2((x))
b2 --> y2((y))
x2 --> a2[a]
x2 --> bl2[bl]
y2 --> br2[br]
y2 --> c2[c]
end

subgraph case2
x1((x)) --> a1[a]
x1 --> y1((y))
y1 --> b1((b))
y1 --> c1[c]
b1 --> bl[bl]
b1 --> br[br]
end
{% endmermaid %}



# Parameters' Range

看上去十分自然，但是我们需要保证旋转后的WB-tree是平衡的，换言之我们需要对 $\Delta$ 和 $\alpha$ 做出限制，具体来说需要求解下面的约束条件：

- 旋转前 $P_{\text{before}}$：
  - 插入（删除）前 WB-tree 是平衡的
  - 插入（删除）后 WB-tree 不平衡，而左右子树是平衡的
  - 右子树满足单旋（双旋）条件
- 旋转后 $P_{\text{after}}$：
  - 整颗子树是平衡的
  - 左右子树是平衡的

很明显 $\alpha$ 是依赖 $\Delta$ 的参数，所以我们需要保证 ：$P_{\text{before}} \Rightarrow P_{\text{after}}$  。

看上去这个问题貌似不复杂，但是这个解的范围要比想象中复杂的多。最早1972年论文中给出一组解 $(1+\sqrt{2}, \sqrt{2})$， 1992年的论文给出一个解的范围，后续Haskell和Scheme都基于此实现了自己的`Map`。不幸的是，1992年的论文是错误的，有人给GHC提交了Bug，当时`Data.Map`使用的是 $(5, 2)$ 这对参数，后续修改为了 $(4,2)$ ，今天使用 $(3,2)$ 修复了这个Bug。后来MIT Scheme也发现了Bug，当时其使用 $(5,1)$。直到2010年的论文才给出了这个问题的确定范围，确定了有且仅有 $(4,2)$ 和 $(3,2)$ 这两组整数解（原论文证明的是原始WB-tree，可以转化为我们这个版本的定义），并且说明了1972年的解就是“最优”的解（不过因为其使用无理数在实际应用中未采用）。

简单总结，这个问题是很麻烦的，详细请读者自行阅读2010年的论文，结论很简单，整数解有且仅有 $(4,2)$ 和 $(3,2)$。在实际应用中，$\Delta$ 越小，越平衡，插入的效率越高；$\Delta$ 越大，删除的效率越高。

# Code

```haskell
{-# LANGUAGE BangPatterns #-}
module WBT where

import Prelude hiding (elem)

data Tree a = Lf | Br !Int (Tree a) a (Tree a) deriving (Show)

size :: Tree a -> Int
size Lf = 0
size (Br s _ _ _) = s

br :: Tree a -> a -> Tree a -> Tree a
br l x r = Br (1 + size l + size r) l x r

elem :: (Ord a) => a -> Tree a -> Bool  
elem _ Lf = False 
elem x (Br _ l y r) = case x `compare` y of 
    LT -> elem x l
    GT -> elem x r
    EQ -> True

rank :: (Ord a) => a -> Tree a -> Int
rank v = go 0
    where go !acc Lf = acc
          go !acc (Br _ l x r) = case v `compare` x of 
              LT -> go acc l
              _  -> go (acc + size l + 1) r

index :: Int -> Tree a -> a
index = go
    where go !n t | size t < n = error "there is no enough elements"
          go !n (Br _ l x r) 
            | n <= size l     = go n l
            | n == size l + 1 = x
            | otherwise       = go (n - 1 - size l) r


delta, alpha :: Int
delta = 4
alpha = 2

balance :: Tree a -> a -> Tree a -> Tree a
balance l x r
    | size l + size r <= 1    = br l x r
    | size r > size l * delta = rotateL l x r
    | size l > size r * delta = rotateR l x r
    | otherwise               = br l x r

rotateL, rotateR :: Tree a -> a -> Tree a -> Tree a
rotateL a x (Br _ b y c)
    | size b < size c * alpha = br (br a x b) y c
    | otherwise = case b of Br _ l v r -> br (br a x l) v (br r y c)
rotateR (Br _ a x b) y c
    | size b < size a * alpha = br a x (br b y c)
    | otherwise = case b of Br _ l v r -> br (br a x l) v (br r y c)

insert :: (Ord a) => a -> Tree a -> Tree a
insert v = ins 
    where ins Lf = br Lf v Lf
          ins t@(Br _ l x r) = case v `compare` x of 
              LT -> balance (ins l) x r
              GT -> balance l x (ins r)
              EQ -> t 

delete :: (Ord a) => a -> Tree a -> Tree a
delete v = del 
    where del Lf = Lf
          del t@(Br _ l x r) = case v `compare` x of 
              LT -> balance (del l) x r
              GT -> balance l x (del r)
              EQ -> join l r

join :: (Ord a) => Tree a -> Tree a -> Tree a
join Lf t = t
join t Lf = t
join l r = let x = maxElem l in balance (delete x l) x r

maxElem :: (Ord a) => Tree a -> a
maxElem Lf = error "try to find max element in an empty tree"
maxElem (Br _ _ x Lf) = x
maxElem (Br _ _ _ r) = maxElem r

instance Foldable Tree where
    foldMap f Lf = mempty 
    foldMap f (Br _ l x r) = foldMap f l `mappend` f x `mappend` foldMap f r

toList :: Tree a -> [a]
toList = foldr (:) []

fromList :: (Foldable t, Ord a) => t a -> Tree a
fromList = foldr insert Lf 

---

isBalanced :: Tree a -> Bool
isBalanced Lf = True
isBalanced (Br _ l x r) 
    | size l + size r <= 1 = True 
    | otherwise = size l <= size r * delta
                && size r <= size l * delta
                && isBalanced l && isBalanced r
```

