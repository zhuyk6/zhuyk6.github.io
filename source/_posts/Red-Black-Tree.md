---
title: Red Black Tree
tags: [RB tree, Balanced tree]
mathjax: true
date: 2021-08-19 10:29:43
categories:
- Data Structures
---

本文旨在用最简单、最直观的逻辑、实现来讲清楚Red-Black tree的理论和实现。

参考资料：

- [wiki](https://en.wikipedia.org/wiki/Red%E2%80%93black_tree)
- [Brilliant wiki](https://brilliant.org/wiki/red-black-tree/)
- [Abhiroop Sarkar's Blog](https://abhiroop.github.io/Haskell-Red-Black-Tree/?utm_source=pocket_mylist)
- [Matt Might's Blog](https://matt.might.net/articles/red-black-delete/?utm_source=pocket_mylist)



<!--more-->



# Overview

Red-Black tree （RB-tree）顾名思义，就是在Binary Search Tree（BST）的基础上，每个内部数据节点附带一个“颜色”信息，颜色只能是“Red”红色或者“Black”黑色，并且满足下面三条性质：

1. root和leaf节点必须是Black；
2. Red节点的“**父亲**”必须是Black；
3. 对于任意一个节点，从该节点出发到其所有leaf的黑色路径长度相等。

![RB-tree example](Red-black_tree_example.svg.png)

仔细品品这三条性质：

1. 没什么好说的，算是一种规整化；
2. 任意父子不能同时为红色，也就是说Red不能相邻；
3. 有很多种等价描述，也可以描述为：
   - 每个Leaf的Black-Depth（从root到该节点的黑色节点数目）相等；
   - 对于每个节点，其两个儿子的Black-Height相等；

表面上看上去，会觉得RB-tree的定义十分“不自然”，为什么要定义貌似屁用没有的“颜色”？颜色的实际意义是什么？2、3的约束为什么能保证树高？没有红色节点行不行？



# Red-Black Tree Height

对于平衡树，关键就是对于树高的控制，RB-tree是怎么控制树高的，关键就是性质2、3，下面证明：

**RB-tree和4阶B-tree是同构的**。

{% note info  %}
4阶B-tree每个内部节点存储1..3个值 并且 拥有2..4个儿子
{% endnote %}

下面说明RB-tree可以通过压缩Red节点形成同构的B-tree：

- 如果一个Black节点没有Red儿子，保持不变，拥有2个儿子；
- 如果Black节点有一个Red儿子，将Red吸收，并将其儿子作为自己的儿子，拥有3个儿子；
- 如果Black节点有2个Red儿子，将Red吸收，并将其儿子作为自己的儿子，拥有4个儿子。

以下图为例：

![Before Merge](befor_merge.png)

将Red节点merge后：

![After Merge](after_merge.png)

下图是上一节中example的等价B-tree。

![RB-tree and B-tree](Red-black_tree_example_(B-tree_analogy).svg.png)



其实这也就是RB-tree的本质，就是4阶B-tree，只不过用等价的2叉树方便编程罢了，~~此文终结。~~ 不过该证明的还是要证明，本文适用于没学过B-tree的小白。



下面证明 $n$ 个节点（不含Leaf）的RB-tree的树高 $h\leq 2 \log_2(n+1)$：

首先设RB-tree的等价B-tree树高为 $h_1$，根据性质2，很容易得到

$$h_1\geq h / 2$$

merge前后Leaf一定被保留，$n$ 节点的BST拥有 $n+1$ 个Leaf，根据性质3，merge后得到的树一定高度平衡（所有Leaf拥有相同的深度），所以

$$\begin{align} n+1 & \geq 2^{h_1} \\
\log_2(n+1) & \geq h_1 \geq h / 2 \\
h & \leq 2 \log_2(n+1)
\end{align}$$



# Insertion

和一般的BST插入相同，在Leaf位置插入一个Red节点，然后调整平衡。

```haskell
insert :: (Ord a) => a -> RBTree a -> RBTree a
insert v t = blacken $ ins t
    where   ins E = T R E v E
            ins (T c l x r) = case compare v x of
                LT -> balance c (ins l) x r
                EQ -> T c l x r 
                GT -> balance c l x (ins r)
```

为什么插入Red节点？是为了不破坏性质3，这样插入只可能破坏性质2。

对于性质2的破坏（出现连续Red），显然有且仅有4种情况：左左、左右、右左、右右。

```haskell
data Color = R | B deriving (Show, Enum)
data RBTree a = E | T Color (RBTree a) a (RBTree a)

balance :: Color -> RBTree a -> a -> RBTree a -> RBTree a

balance B (T R (T R a x b) y c) z d = T R (T B a x b) y (T B c z d)
balance B (T R a x (T R b y c)) z d = T R (T B a x b) y (T B c z d)
balance B a x (T R (T R b y c) z d) = T R (T B a x b) y (T B c z d)
balance B a x (T R b y (T R c z d)) = T R (T B a x b) y (T B c z d)

balance c l x r = T c l x r
```

{% note  %}
这里又要大吹特吹Haskell支持模式匹配的好处了。

 BST的定义是什么：中缀有序。所以我们将Tree定义写成中缀形式，这样一来从左向右就是有序的，再次说明了“代码即数据”的优点。

 Rotate？根本不需要好吧，因为只要满足“中缀有序”的性质，括号的位置随便加，所以rotate本质就是调整括号位置。

 这里我们利用模式匹配，人为将key写成`x`、`y`、`z`，子树写成`a`、`b`、`c`、`d`，显然只需要保证平衡前后他们的相对位置不变，BST的性质就没变。

{% endnote %}


{% note warning %}
需要注意：对于RB-tree的性质3，要保证平衡前后根节点的Black-Height不变，自己手动画一画图就知道了。
{% endnote %}


# Deletion

相比插入，删除操作就复杂的太多了，因为会同时破坏性质2和性质3，所以可能的情况就非常之多，参考[wiki](https://en.wikipedia.org/wiki/Red%E2%80%93black_tree)，简直没有任何美感，单纯的暴力的讨论各种可能的情况。

如果本文也是像wiki那样讨论分析delete的各种情况，那就可以去死了。

事实上，对于删除操作，存在一种非常巧妙trick，可以将删除的复杂度化简到和插入一样简单，参考[Matt Might's Blog](https://matt.might.net/articles/red-black-delete/?utm_source=pocket_mylist)。

首先思考一个问题，为啥删除要比插入复杂？或者说为啥插入就那么简单？

因为我们只插入Red节点，不会破坏性质3，只会破坏性质2。

同理，我们能否使用相似的策略，使得性质3不被破坏。表面上看貌似不可能，删除Red节点好说，但是如果删除一个Black节点呢？

尝试换一种视角看待性质3：Red贡献0，Black贡献+1，每个节点的两个儿子具有相同的“加权高度”。

我们删除一个Black节点，这颗子树的加权高度-1，所以可以通过将其父亲的权重+1来维持性质3。

非常有意思，相当于我们引入新的“颜色”：Double Black（+2）。显然这只是临时的颜色，最终的RB-tree我们肯定不希望看见这东西，所以需要引入新的操作`bubble`：如果一个节点的儿子是BB，那么将“权重上调”。所谓“权重上调”，就是将两个儿子的颜色减轻，自己的颜色加深。

```haskell
bubble :: Color -> RBTree a -> a -> RBTree a -> RBTree a
bubble c l x r
    | isBB l || isBB r = balance (blacker c) (redder' l) x (redder' r)
    | otherwise        = balance c l x r
```

易知，`bubble`前的非法状态有且仅有以下6种：（图中“灰色”代表Double Black）

![Before Bubble](before_bubble.png)

需要说明：

- 为什么没有两个儿子都是Double Black？因为只可能delete的分支传上来一个BB。
- 为什么没有父亲兄弟都是Red的情况？因为delete前RB-tree是合法的，这样违背性质2。



这6种情况`bubble`后的局面是：(White代表Negative Black(-1) )

![After Bubble](after_bubble.png)

- 对于左边两种情况，和插入的非法情况相似，可能导致Red连续，Red连续的情况有4种；

- 对于中间两种情况，可能导致之前插入的非法情况，Red连续有4种；

- 对于右边两种情况，兄弟的“权重下调”，因为兄弟本身就是Red (+0)，下调后是Negative Black(-1)，产生了一种新的颜色Negative Black，显然这东西也不是我们希望看到的，所以也要消除：

  ![](balance_3.jpeg)

  如上图，因为`y`是NB，是Red变来的，所以`y`的两个儿子一定都是Black，不妨选取左儿子，然后平衡。需要注意的是，因为`c`的Black-Height是`n+1`和`a`、`b`、`d`不同，所以需要对`c`进行降级，降级后继续平衡。

综上所述，`balance`总共有 $4 + 4 + 2 = 10$ 种情况。

```haskell
balance :: Color -> RBTree a -> a -> RBTree a -> RBTree a

balance B (T R (T R a x b) y c) z d = T R (T B a x b) y (T B c z d)
balance B (T R a x (T R b y c)) z d = T R (T B a x b) y (T B c z d)
balance B a x (T R (T R b y c) z d) = T R (T B a x b) y (T B c z d)
balance B a x (T R b y (T R c z d)) = T R (T B a x b) y (T B c z d)

balance BB a x (T R b y (T R c z d)) = T B (T B a x b) y (T B c z d)
balance BB a x (T R (T R b y c) z d) = T B (T B a x b) y (T B c z d)
balance BB (T R (T R a x b) y c) z d = T B (T B a x b) y (T B c z d)
balance BB (T R a x (T R b y c)) z d = T B (T B a x b) y (T B c z d)

balance BB a x (T NB b y (T B c z d)) = T B (balance B a x (redder' b)) y (T B c z d)
balance BB (T NB (T B a x b) y c) z d = T B (T B a x b) y (balance B (redder' c) z d)

balance c l x r = T c l x r
```

讨论完了`bubble`和`balance`，最后回到`delete`。按照一般的BST删除方法，首先找到该节点，然后删除它：

- `T R E _ E`：红色节点直接删除返回一个`E`即可；
- `T B E _ E`：黑色节点删除导致Black-Height减少，返回`EE`，相当于Double Black的Leaf；
- 删除节点是黑色，其儿子一个是Red，一个是`E`，将儿子修改为Black返回；
- 左右儿子都不是`E`，将节点值替换为左儿子的最大值，并在左儿子中删除它。

```haskell
delete :: (Ord a) => a -> RBTree a -> RBTree a
delete v t = blacken $ del t
    where
        del E = E
        del t@(T c l x r) = case compare v x of 
            LT -> bubble c (del l) x r 
            GT -> bubble c l x (del r)
            EQ -> remove t

remove :: RBTree a -> RBTree a
remove (T R E _ E) = E
remove (T B E _ E) = EE
remove (T B E _ (T R l x r)) = T B l x r
remove (T B (T R l x r) _ E) = T B l x r
remove (T c l x r) = bubble c l' x' r
    where
        l' = removeMax l
        x' = findMax l

removeMax :: RBTree a -> RBTree a 
removeMax t@(T _ _ _ E) = remove t
removeMax (T c l x r) = bubble c l x (removeMax r)

findMax :: RBTree a -> a
findMax E = error "findMax in an empty tree"
findMax (T _ _ x E) = x
findMax (T _ _ _ r) = findMax r
```

# Code

```haskell
module RBTree where

import Text.PrettyPrint
import Prelude hiding (lookup)

data Color = NB | R | B | BB deriving (Show, Enum)

data RBTree a = E | EE | T Color (RBTree a) a (RBTree a)

blacker :: Color -> Color
blacker = succ

redder :: Color -> Color
redder = pred

blacker' :: RBTree a -> RBTree a
blacker' EE = error "blacker' a EE node"
blacker' E = EE
blacker' (T c l x r) = T (blacker c) l x r

redder' :: RBTree a -> RBTree a
redder' E = error "redder' a E node"
redder' EE = E
redder' (T c l x r) = T (redder c) l x r

isBB :: RBTree a -> Bool 
isBB EE = True 
isBB (T BB _ _ _) = True 
isBB _ = False

blacken :: RBTree a -> RBTree a
blacken (T _ l x r) = T B l x r
blacken _ = E

redden :: RBTree a -> RBTree a
redden (T _ l x r) = T R l x r


balance :: Color -> RBTree a -> a -> RBTree a -> RBTree a

balance B (T R (T R a x b) y c) z d = T R (T B a x b) y (T B c z d)
balance B (T R a x (T R b y c)) z d = T R (T B a x b) y (T B c z d)
balance B a x (T R (T R b y c) z d) = T R (T B a x b) y (T B c z d)
balance B a x (T R b y (T R c z d)) = T R (T B a x b) y (T B c z d)

balance BB a x (T R b y (T R c z d)) = T B (T B a x b) y (T B c z d)
balance BB a x (T R (T R b y c) z d) = T B (T B a x b) y (T B c z d)
balance BB (T R (T R a x b) y c) z d = T B (T B a x b) y (T B c z d)
balance BB (T R a x (T R b y c)) z d = T B (T B a x b) y (T B c z d)

balance BB a x (T NB b y (T B c z d)) = T B (balance B a x (redder' b)) y (T B c z d)
balance BB (T NB (T B a x b) y c) z d = T B (T B a x b) y (balance B (redder' c) z d)

balance c l x r = T c l x r

bubble :: Color -> RBTree a -> a -> RBTree a -> RBTree a
bubble c l x r
    | isBB l || isBB r = balance (blacker c) (redder' l) x (redder' r)
    | otherwise        = balance c l x r

insert :: (Ord a) => a -> RBTree a -> RBTree a
insert v t = blacken $ ins t
    where   ins E = T R E v E
            ins (T c l x r) = case compare v x of
                LT -> balance c (ins l) x r
                EQ -> T c l x r 
                GT -> balance c l x (ins r)

delete :: (Ord a) => a -> RBTree a -> RBTree a
delete v t = blacken $ del t
    where
        del E = E
        del t@(T c l x r) = case compare v x of 
            LT -> bubble c (del l) x r 
            GT -> bubble c l x (del r)
            EQ -> remove t

remove :: RBTree a -> RBTree a
remove (T R E _ E) = E
remove (T B E _ E) = EE
remove (T B E _ (T R l x r)) = T B l x r
remove (T B (T R l x r) _ E) = T B l x r
remove (T c l x r) = bubble c l' x' r
    where
        l' = removeMax l
        x' = findMax l

removeMax :: RBTree a -> RBTree a 
removeMax t@(T _ _ _ E) = remove t
removeMax (T c l x r) = bubble c l x (removeMax r)

findMax :: RBTree a -> a
findMax E = error "findMax in an empty tree"
findMax (T _ _ x E) = x
findMax (T _ _ _ r) = findMax r

lookup :: (Ord a) => a -> RBTree a -> Maybe a
lookup _ E = Nothing 
lookup a (T _ l x r) = case compare a x of 
    LT -> lookup a l
    GT -> lookup a r
    EQ -> Just x

---

showTree :: (Show a) => RBTree a -> Doc 
showTree E = text "E"
showTree (T c l x r) = 
    text "T" <+> text (show c) <+> text (show x)
    $+$ (text "  " <+> vcat [showTree l, showTree r]) 

instance (Show a) => Show (RBTree a) where
    show = render . showTree

instance Foldable RBTree where
    foldMap f E = mempty
    foldMap f (T _ l x r) = foldMap f l `mappend` f x `mappend` foldMap f r

toList :: RBTree a -> [a]
toList = foldr (:) []

fromList :: (Foldable t, Ord a) => t a -> RBTree a
fromList = foldr insert E 
```



# Conclusion

回过头来看看RB-tree，会发现这东西是如此简单，如此自然。

我们知道了RB-tree和4阶B-tree是同构的，只不过是B-tree的二叉树表示罢了，Red节点不过就是B-tree的内部节点，所以熟悉B-tree的话可以将插入删除都类比B-tree学习。

对比另一个著名的平衡树AVL，我们能得到什么结论。众所周知，AVL子树高度差不超过2，所以更加“平衡”，RB-tree的子树高度差可能达到一倍之多，所以AVL的高度是严格 $\log (n+1)$，而RB-tree的高度可能达到 $2\log(n+1)$。因为高度更低，所以AVL的查询效率更高（理论上最高），但是修改的时候也就更容易破坏结构，需要更多的平衡操作。RB-tree由于其限制更加宽松，只会在Red相邻时平衡，所以常数要小的多。

下面从更深层次来思考平衡树问题。

对于绝对平衡的树（所有叶子具有相同深度），显然查询效率一定是最高的，但是它甚至不能够修改，对于2叉满的绝对平衡树，高度为 $h$，其节点数目为 $2^0 + 2^1 + \cdots + 2^{h-1} = 2^h - 1$。

所以我们需要 “放宽松” 限制条件，很直观的就有两种思路：

- 放松高度、size等平衡标准的限制：高度差不超过 $\Delta$，$size_L \leq \alpha \times size_R$ 等等；
- 放松树结构的限制：允许多叉树结构。

本质上，都是引入了“缓冲区”的概念，~~计算机科学的大多数问题都可以通过加cache解决。~~ 

对于放松树结构是最好理解的，2-3 tree是很标准的平衡树结构，但是和AVL一样太过于标准，轻易修改就会导致结构非法，基于2-3 tree的Finger Tree如果允许1、4合法，相当于增加的缓冲区，就会大大减少对于树结构的修改。

对于放松平衡标准也差不多一个意思，AVL动不动就需要平衡，RB-tree只有在Red连续才需要，Size-Balanced tree只有在左右子树平衡因子超过阈值才平衡，显然这个平衡到非法之间的容错空间就相当于缓冲区。

当然世上没有十全十美的事情，有些事物天生就是对立的。标准越宽松，结构的改变次数就越少，但是与此同时树也可能越不平衡。所以在实际应用中，要benchmark，要看操作的占比，如果查询远超过修改，显然AVL是最合适的；如果修改远超过查询，对于`insert`和`delete`的时间复杂度取决于树高和平衡限制，这之间的平衡点显然取决于实际数据。另一方面，对于结构的控制越多，代码复杂度越高，程序的常数也就越大，Haskell的`Set`和`Map`使用Size-Balanced tree就是因为其实现极其简单并且常数很小。

No silver bullet.
