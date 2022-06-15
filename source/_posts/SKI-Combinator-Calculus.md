---
title: SKI Combinator Calculus
tags: [SKI]
mathjax: false
date: 2021-08-02 00:12:57
categories:
---

本文借助 Codewars 一道题目 [Sure, but can you SKI? (I)](https://www.codewars.com/kata/5a02dccf32b8b988120000da/haskell) 来简单学习 SKI。

参考 [wiki](https://en.wikipedia.org/wiki/SKI_combinator_calculus)。

<!--more-->

# SKI

SKI 组合子计算 是一个计算系统，和lambda演算一样，是图灵完备的。

SKI 顾名思义，由三种基本组合子构成，计算规则如下：

- `I x = x`
- `K x y = x`
- `S x y z = x z (y z)`

很明显，`I`就是`id :: a -> a`，`K`就是`const :: a -> b -> a`，只有`S`看似复杂一些，`S`用来实现“函数应用”。

> 需要指出，`S K _ x = K x (_ x) = x = I x`，所以可以使用 `SKK` 来代替 `I`，可见 `I` 不是必须的，只是一个语法糖，所以SKI也称作SK演算。


# 与lambda演算转换

SKI 演算只使用这三种组合子，所以是一种组合子风格的计算系统。将lambda演算转换为SKI，关键就是将lambda演算写成point-free风格，消去参数。

显然，这种转换并不是唯一的，或许有必要追求所谓最短转换，不过在此仅介绍一种通用的转换方法：

- `\x -> x`：写作`I`
- `\x -> A`：其中`A`不含`x`，直接写作`K A`;
- `\x -> A x`：其中`A`不含`x`，直接写作`A`;
- `\x -> A B`：转化为`\x -> S (\x -> A) (\x -> B) x`。

按照这种通用思路，可以将任意lambda演算转换为SKI，不过这种方法的转化可能是指数增长的，非常可怕。

下面举一些例子来说明：

```haskell
rev :: a -> (a -> b) -> b
-- \a f -> f a 
-- \a f -> f (K a f)
-- \a f -> (I f) (K a f)
-- \a f -> S I (K a) f
-- \a -> S I (K a)
-- \a -> (S I) (K a)
-- \a -> (K (S I) a) (K a)
-- \a -> S (K (S I)) K a
rev = s (k (s i)) k
```

```haskell
comp :: SKI ((b -> c) -> (a -> b) -> (a -> c))
-- \g f -> g . f 
-- \g f x -> g (f x)
-- \g f x -> (K g x) (f x)
-- \g f x -> S (K g) f x 
-- \g f -> S (K g) f
-- \g -> S (K g)
-- \g -> (K S g) (K g)
-- \g -> S (K S) K g
-- S (K S) K
comp = S (K S) K
```

```haskell
flip' :: SKI ((a -> b -> c) -> (b -> a -> c))
-- -- \f -> flip f
-- -- \f b a -> f a b
-- -- \f b a -> (f a) b 
-- -- \f b a -> (f a) (K b a)
-- -- \f b a -> S f (K b) a 
-- -- \f b -> S f (K b)
-- -- \f b -> (K (S f) b) (K b)
-- -- \f b -> S (K (S f)) K b
-- -- \f -> S (K (S f)) K
-- -- \f -> ((S . K . S) f) (K K f)
-- -- \f -> S (S . K . S) (K K) f 
-- -- S (S . K . S) (K K)
-- flip' = S (comp S (comp K S)) (K K)

-- \f b a -> f a b 
-- \f b a -> (f a) (K b a)
-- \f b -> S f (K b)
-- \f -> (S f) . K 
-- \f -> comp (S f) K
-- \f -> (comp comp S f) (K K f)
-- S (comp comp S) (K K)
-- flip' = S (comp comp S) (K K)

-- \f b a -> f a b 
-- \f b a -> (f a) (K b a)
-- \f b -> S f (K b)
-- \f b -> comp (S f) K b
-- \f -> comp (S f) K
-- \f -> (K comp f) (S f) (K K f)
-- \f -> S (K comp) S f (K K f)
-- S (S (K comp) S) (K K)
flip' = S (S (K comp) S) (K K)
```

> 从`flip`就可以看出来，同一个lambda表达式可以有非常多等价的SKI表达，不同的抽象方法可能得到长得完全不一样的表达式， ~~这或许就是优化空间，笑~~

```haskell
rotr :: SKI (a -> (c -> a -> b) -> c -> b)
-- \a f c -> f c a 
-- \a f c -> (f c) (K a c)
-- \a f c -> S f (K a) c 
-- \a f -> S f (K a)
-- \a f -> (S f) (K (K a) f)
-- \a f -> S S (K (K a)) f 
-- \a -> S S (K (K a))
-- \a -> (K (S S) a) ((K . K) a)
-- S (K (S S)) (K . K)
-- rotr = S <@> (K <@> (S <@> S)) <@> (comp <@> K <@> K)

-- \a f c -> f c a 
-- \a f c -> flip' f a c
-- \a f -> flip' f a
-- \a f -> flip' flip' a f
-- flip' flip'
rotr = flip' flip'
```

```haskell
rotv :: SKI (a -> b -> (a -> b -> c) -> c)
-- \a b f -> f a b 
-- \a b f -> (f a) (K b f)
-- \a b f -> (rev a f) (K b f)
-- \a b f -> S (rev a) (K b) f 
-- \a b -> S (rev a) (K b) 
-- \a b -> (K (S (rev a)) b) (K b)
-- \a -> S (K (S (rev a))) K
-- \a -> ((S . K . S . rev) a) (K K a)
-- S (S . K . S . rev) (K K)

-- \a b f -> f a b 
-- \a b f -> rev a f b
-- \a b f -> flip' (rev a) b f 
-- \a -> flip' (rev a)
-- comp flip' rev
rotv = comp flip' rev 
```

```haskell
join :: SKI ((a -> a -> b) -> a -> b)
-- join = Ap (Ap flip' S) I
-- \f a -> f a a
-- \f a -> (f a) (I a)
-- \f -> S f I 
-- \f -> flip' S I f

-- \f a -> f a a 
-- \f a -> (f a) (S K f a)
-- \f -> S f (S K f)
-- S S (S K)
join = S  S  (S  K)
```

上面几个例子都是用于控制函数参数位置的组合子，使用他们可以更方便的控制更复杂的结构。

下面以`Bool`类型为例子，说明使用SKI实现lambda演算版本的布尔类型。

首先介绍Church encoding版本的`Bool`类型：

```haskell
type Bool' a = a -> a -> a

true :: Bool' a
true a b = a

false :: Bool' a
false a b = b
```

可以看出，Church encoding版本的`Bool`类型本质上是位置选择函数，选择第一个位置还是第二个，所以我们可以借助上面实现的各种参数位置的控制函数来实现下面的函数。

当然这是因为任何一个二元类型`A`的计算`A -> r`都同构于`r -> r -> r`，这放到同构和Church encoding专题再详细讨论。

```haskell
true :: SKI (Bool' a)
true = K
-- \a b -> a

false :: SKI (Bool' a)
false = K I
-- \a b -> b

not' :: SKI (Bool' a -> Bool' a)
not' = flip'
-- \b -> flip b

and' :: SKI (Bool' (Bool' a) -> Bool' a -> Bool' a)
-- and' = Ap (Ap S S) (Ap K (Ap K false))
-- \x y -> x y F
-- \x y -> (x y) (K F y)
-- \x -> S x (K F)
-- \x -> (S x) (K (K F) x)
-- S S (K (K F))

-- \x y -> x y F 
-- \x y -> rotr F x y
-- rotr F
and' = rotr false


or' :: SKI (Bool' (Bool' a) -> Bool' a -> Bool' a)
-- or' = Ap (Ap S I) (Ap K true)
-- \x y -> x T y
-- \x -> x T
-- \x -> (I x) (K T x)
-- S I (K T)

-- \x y -> x T y 
-- \x y -> rev T x y
-- rev T 
or' = rev true


xor' :: SKI (Bool' (Bool' a -> Bool' a) -> Bool' a -> Bool' a)
-- xor' = Ap (Ap S x) y
--     where 
--         comp' = Ap flip' comp
--         x = Ap (Ap comp S) (Ap comp' not')
--         y = Ap K I

-- \x y -> x (not y) y 
-- \x y -> (comp x not) y (I y)
-- \x -> S (comp x not) I
-- \x -> (comp S (\x -> comp x not)) x I
-- \x -> (comp S (comp' not)) x (K I x)
-- S (comp S (comp' not)) (K I)

-- \x y -> x (not y) y
-- \x y -> comp x not y y
-- \x -> join (comp x not)
-- \x -> join (flip' comp not x)
-- comp join (flip' comp not)
-- xor' = comp  join  (flip'  comp  not')

-- \x -> x not I
-- \x -> rev I (x not)
-- \x -> rev I (rev not x)
-- comp (rev I) (rev not)
xor' = comp (rev  I)  (rev not')
```

比较有意思的是，根据不同层次的抽象，写出来的代码复杂度千差万别。

如果只是简单的将`true`和`false`当作两个位置选择函数，然后每次将完整的参数全部带入然后化简，比如`and = \x y a b -> x (y a b) b`，那就太复杂了，实际上借助我们对布尔运算的先验知识，可以很容易写出下面的等价描述：

- `and = \x y -> x y F`
- `or = \x y -> x T y`
- `xor = \x y -> x (not y) y`

有趣的是，还可以借助先验知识继续化简：

- `and = \x -> x I (K F)`
- `or = \x -> x T I`
- `xor = \x -> x not I`

而这两者之间的互相转换，仅仅只有SKI的信息应该是无法完成的，换言之，这二者的等价依赖`Bool'`这个类型。（~~或许我太菜了没看出来~~）

# SKI 的意义与价值

不懂，或许未来就懂了，再回来填坑。

目前来看，SKI是一个足够小的规则集，是图灵完备，问题是这玩意儿和lambda演算比有啥优势吗，人家lambda演算也只有两条基本规则阿，而且比SKI方便表达多了。或许主要魅力在于“组合子”？point-free？ ~~不懂，有屁用阿……~~