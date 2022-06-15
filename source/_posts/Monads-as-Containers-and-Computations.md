---
title: Monads as Containers and Computations
date: 2021-06-25 11:11:16
categories:
- Languages
- Haskell
tags: [monad]
mathjax:
---

[Monads as Containers](https://wiki.haskell.org/Monads_as_containers)

[Monads as Computation](https://wiki.haskell.org/Monads_as_computation)

<!--more-->

Monad作为Haskell中非常核心的一个Typeclass，其概念和意义有很多种理解。本文是[Haskell wiki](https://wiki.haskell.org/HaskellWiki:Community)上的两篇文章[《Monads as Containers》](https://wiki.haskell.org/Monads_as_containers)和[《Monads as Computation》](https://wiki.haskell.org/Monads_as_computation)的笔记。



# Monad as Containers

“容器”这种理解是科普时经常采用的，也是学习过程中最早期建立的理解。这种理解和`Monad`的定义有关：

## Monad 定义

```haskell
class Functor f where
    fmap :: (a -> b) -> f a -> f b

class (Functor m) => Monad m where
    return :: a -> m a
    (>>=) :: m a -> (a -> m b) -> m b
    join :: m (m a) -> m a
```

对于任意一个`Monad m`，`m`首先要是一个`Functor`。

{% note info %}
  其实在`Functor`和`Monad`中间还夹着一个`Applicative`，不过一方面历史原因`Applicative`出现较晚，导致标准库并没有这种约束，另一方面学习理解`Monad`暂时还不需要`Applicative`，这二者的详细讨论放在将来。
{% endnote %}

什么是`Functor`？最简单直观的理解就是一个**容器**，`f a`就是里面放着类型为`a`的数据，容器类型为`f`的这么一个东西。`Functor`提供了一个接口`fmap`，其含义是将一个普通的`a->b`函数`lift`成为`f a -> f b`的函数，也可以认为是将容器外部的函数“包装”成了容器函数。可以看出，很多数据结构天然就是`Functor`。

{% mermaid graph LR %}
    A[a] --> B[fmap f]--> C[b]
{% endmermaid %}

那什么是`Monad`？用“容器”的观点，就是支持更多操作的**更强**的容器：

- `fmap :: (a -> b) -> m a -> m b`：把`a -> b`的函数“提升”为`m a -> m b`的函数
- `return :: a -> m a`：就是把一个数据`a`包装成`m a`
- `join :: m (m a) -> m a`：把嵌套的容器“**合并**”成一个，flatten

{% note info %}
  这里将`join`当作`Monad`基本操作，而没有选取`(>>=)`，实际上两者是等价的可以转换，不过在范畴论里`join`是`Monad`定义的一部分。
  `join`的理解是非常关键的，注意是“合并”，并不是简单的将外层容器去掉。
{% endnote %}

{% mermaid graph LR %}
subgraph bind
    A[a] --> B>a -> m b]
    B --> C[b]
end
{% endmermaid %}

上面三种操作就是`Monad`定义的全部了，我们可以根据它们构造出更多工具。

首先是`(>>=)`：

```haskell
(>>=) :: (Monad m) => m a -> (a -> m b) -> m b
xs >>= f = join (fmap f xs)
```

`(>>=)`英文名称叫bind，不知道应该怎么翻译。用“容器”的观点，就是将`m a`里面的数据`a`取出，放到`a -> m b`的函数`f`里。其实现就是将`f`升格为`m a -> m (m b)`，最后用`join`将多余的`m` flatten。

标准库`Monad`的定义是`return`和`(>>=)`，用这两个可以导出`liftM`和`join`：

```haskell
liftM :: (Monad m) => (a -> b) -> m a -> m b
liftM f xs = xs >>= (return . f)

join :: (Monad m) => m (m a) -> m a
join xss = xss >>= id
```

`Monad`的基本定义就是这些，事实上一个定义良好的`Monad`还应该遵守三条单子律，不过在这里就不讨论了，放在后面Monads as Computation讨论。

## Monad examples

前面提到过，对于初学者其实将`Monad`视作“容器”是很好理解的。但是，这种观点继续下去问题就大了，对于那些具体的Monad instance，这些“容器”是什么？

### `Identity a`

```haskell
newtype Identity a = Identity { runIdentity :: a }

instance Functor Identity where
    fmap f (Identity a) = Identity (f a)

instance Monad Identity where
    return = Identity
    (Identity a) >>= f = f a
```

`Identity`这个类型，在类型中的地位相当于`id`在函数中的地位，就是什么都不做，直接封装。



### `Maybe a`

```haskell
data Maybe a = Nothing | Just a

instance Functor Maybe where
    fmap _ Nothing = Nothing
    fmap f (Just a) = Just (f a)

instance Monad Maybe where
    return = Just
    Nothing >>= _ = Nothing
    (Just a) >>= f = f a
```

`Maybe`这个类型代表了**可能为空**的容器，空就是`Nothing`，否则就是`Just x`，相当于其他语言中的`None`、`null`的处理。

### `[a]`

```haskell
data List a = Nil | Cons a (List a)

instance Functor List where
    fmap _ Nil = Nil
    fmap f (Cons a as) = Cons (f a) (fmap f as)

instance Monad List where
    return a = Cons a Nil
    Nil >>= _ = Nil
    Cons a as >>= f = f a ++ (as >>= f)
```

`List`这个类型，代表了可能存储零个或者多个数据的容器，相当于数据结构中的单链表。

### `Reader r a`

```haskell
newtype Reader r a = Reader { runReader :: r -> a }

instance Functor (Reader r) where
    fmap f (Reader m) = Reader $ f . m

instance Monad (Reader r) where
    return x = Reader $ \r -> x
    (Reader m) >>= f = f . m
```

从`Reader r`这个类型开始，问题就变得奇怪了起来。复杂的Haskell类型，经常喜欢把一些函数封装成一个类型，这玩意儿也算是“容器”吗？它是怎么把数据`a`存储起来的？

`Reader r`是一个好的开始，毕竟这东西算是最简单的“函数封装成的类型”，用context的理解就是共享一个“只读”变量`r`的计算环境。但是这里我们希望能有一种“容器”视角的理解，怎么把这玩意儿看作一个容器。

关键在于这个`r`，我们将它看作某种index下标，将一个`m :: Reader r a`看作是一个以`r`为下标，每个格子里存放类型`a`的数据的巨大容器。换言之，`runReader :: Reader r a -> r -> a`函数就相当于`lookup :: Eq a => [(a, b)] -> a -> Maybe b `，其作用就是对于一个`Reader`容器获取某个下标里面存放的数据。

也就是说，逻辑上可以将`Reader r a`看作一个巨大无比的数组，当然数组的下标是有限整数，但这里的`r`可以是任意类型，这就是Haskell可怕的地方了，我们得到了一个无限大小的类似数组的容器，并且我们其实并没有为容器中的数据`a`们分配内存空间，每次需要的时候，计算即可。`Reader r a`的代码就相当于里面存储的数据，这就是“代码即数据”的最真切体现了吧。

### `State s a`

```haskell
newtype State s a = State {runState :: s -> (a, s)}

instance Functor (State s) where
    fmap f (State m) = State $ \s -> let (a, s') = m s in (f a, s')

instance Monad (State s) where
    return a = State $ \s -> (a, s)
    m >>= f = State $ \s -> let (a, s') = runState m s
                                (b, t) = runState (f a) s'
                            in (b, t)
```

State Monad可以说是最有意思的Monad之一了，基于State Monad我们可以实现Imperative式的编程。

如果使用computation或者context的视角，理解State Monad是非常简单水到渠成的事情：

- `State s a`封装了一个`s -> (a, s)`的函数，含义就是接受一个状态，返回结果`a`，和下一个状态`s'`
- `State s a`本身是描述这个计算过程，所以每次使用它必须输入一个起始状态`runState m s0`
- `m >>= f`的bind也就很好理解了，输入状态`s`，计算`m`得到中间结果`a`和新状态`s'`，将`a`送给`f `得到新的State Monad，将`s'`送给它得到最终的结果和状态。

没错，State Monad很像一台自动机，其描述了给定某个状态，是怎么转移到下一个状态的，只不过这个自动机不消耗符号，反而每次转移“吐出”一个符号，而且每个节点只有一条出边。

其实如果能够在脑海中想象出这么一张状态转移的图景，那么用“容器”的视角就很简单了。

`State s a`是一个巨大的容器，容器以`s`作为下标，里面存放了数据`a`和下一个节点`s'`。显然`Reader r a`是一个特殊的`State s a`，永远不去修改状态那么状态就是“只读”的。

现在我们去用“容器”的观点理解`join`就水到渠成了：

- `Reader r a`：`join`在巨大数组`mma`里检索`r`获得一个巨大数组`ma`，然后对`ma`检索`r`获得`a`，将`a`放在`r`的位置；
- `State s a`：`join`在巨大数组`mm`里面检索`s`获得一个巨大数组`m`和新的下标`s'`，然后在`m`里检索`s'`获得`a`和又一个下标`t`，最后将`(a, t)`放在下标`s`的位置。

其实从这两个例子也可以看出，为什么前面提到过`join`不是简单的将外层结构去掉，而是“合并”。



将Monad视作存储数据的容器是很简单的观点，但是这个观点抽象层次太低，而且面对复杂的类型，我们很难想象出这个容器到底是怎么存储数据的，`State r a`尚且能够类比成数组或者Map，面对更复杂的`Parser`、`Cont`等等就很麻烦了。



# Monads as Computation

将Monad视作计算本身是更高级的抽象，就像前面说的，一个程序本身是一段代码，这段代码本身就是数据。

## 计算的视角

Monads通常遵循下面几个前提：

- monadic的计算有结果：对于一个Monad `M`，一个类型为`M t`的值就是一段结果类型是`t`的计算；
- 对于任意的值，存在一个计算“什么都不做”，将这个值作为结果：这就是`return :: Monad m => a -> m a`；
- 对于两个计算`x`和`y`，可以将他们组成一个更大的计算`x >> y`：其含义是先计算`x`，抛弃结果，计算`y`，将`y`的结果返回；
- 更进一步，我们允许使用前一个计算的结果来**决定**接下来干什么：这就是`>>=`，`ma>>=f`含义就是先计算`ma`，其结果`a`决定了接下来进行计算`f a`。

{% note info %}
  上面就是计算视角的Monad定义，很容易发现，计算视角有两个很重要的词语“计算”和次序“。

  什么叫“计算”，计算一个`m :: M a`产生类型为`a`的结果，这个计算是指什么？

  “次序”是什么，先计算`x`后计算`y`，这个先后是指什么？

  需要指出的是，因为以这种方式理解Monad涉及到“计算次序”这个东西，所以很容易联想到Imperative Programming，所以误以为Monad的发明就是为了解决lazy computing的
  计算次序不确定性这个问题。事实上这是完全错误的。

  “计算”和“次序”这两个词语，一定要绑定到具体的Monad才有意义，换言之，不同的Monad这两个词语的含义完全不同，这也是FP的魅力，简单的改变可以使同样的句子具有完全不一样
  的含义。

  另一方面，的确可以使用Monad来实现Imperative那样的效果，但这是完全基于函数式和lazy computing的。`Monad`天然不是`IO`，但是`IO`天然是`Monad`。使用Monad来解
  决`IO`是非常漂亮和自然的，`IO`它正好就是Monad，所以可以使用Monad的一众函数库。
{% endnote %}

## Do notation

在进行下面的内容之前，先引入一个语法糖syntax-sugar：

```haskell
do { x } = x
do { x ; <stmts> } = x >> do { <stmts> }
do { v <- x ; <stmts> } = x >>= \v -> do { <stmts> }
do { let <decls> ; <stmts> } = let <decls> in do { <stmts> }
```

这就是do notation，上面的替换发生在编译前期，ghc按照上面的规则将do转换为表达式。

有了这个语法糖，我们可以轻易的写出各种简单漂亮“命令式”风格的代码：

```haskell
main = do
    putStrLn "Enter a line of text:"
    x <- getLine
    putStrLn (reverse x)

fibs = 1 : 1 : [f1 + f2 | (f1, f2) <- zip fibs (tail fibs)]
```


do notation确实是我见过的最漂亮的语法糖了，简单直观好用。

{% note warning %}
 需要注意，虽然do notation看起来非常有“命令式”的特点，但是这并不是命令式。

 ```haskell
 f = do
     x <- f1
     y <- f2
     z <- f3
     return (x, y, z)
 ```

 这段代码如果是命令式，就是依次进行`f1, f2, f3`，将结果打包返回。

 但是前面一节强调过，这段代码的真实**控制流**取决于具体是哪一个Monad，因为do只是一个语法糖，上面代码被转换为`f1 >>= \x -> f2 >>= \y -> f3 >>= \z -> return (x, y, z)`，所以重点是`(>>=)`是怎么实现的，对于`Parser`、`[a]`，可能会发生回溯backtracking，对于更复杂的Monad，例如`Cont`可能发生“中途截断”（类于命令式的`return`），甚至可能会发生并行计算。

 这里就举一个最简单的例子，List。

 ```haskell
 a = do
     x <- [1, 2, 3]
     y <- [4, 5]
     z <- [6]
 --- a = [(1,4,6),(1,5,6),(2,4,6),(2,5,6),(3,4,6),(3,5,6)]
 ```
{% endnote %}

## Monad Laws

Haskell中`Monad`的定义只要求`return `和`(>>=)`这两个函数，理论上我们可以写出各种各样的Monads，但是数学上的单子Monad有着更强的3条性质：

1. `return v >>= f = f v`
2. `x >>= return = x`
3. `(x >>= f) >>= g = x >>= (\v -> f v >>= g)`

直接看数学上美感不足，我们定义`(>=>)`：

```haskell
(>=>) :: (Monad m) => (a -> m b) -> (b -> m c) -> (a -> m c)
f >=> g = \a -> f a >>= g
```

三条单子律可以写作：

1. `return >=> f = f`
2. `f >=> return = f`
3. `(f >=> g) >=> h = f >=> (g >=> h)`

前两条说明了`return`什么都不做，第三条说明了monadic的计算是具有结合律的，可以任意结合，也意味着可以任意拆分为monadic的计算。

{% note danger %}
  需要强调的是，ghc并不会检查这三条单子律，完全由Monad的提供者自己保证。如果你写出的Monad不满足这三条性质，可以通过编译，但是可能会在后续的计算中发生意料之外的错
  误。因为如果不满足单子律，但是使用者还像一般的Monad那样对待它，结果就不可控了，尤其是第三条性质如果不满足，那么我们就不能任意拆分组合。

  最简单的反例就是带有计数器的容器：

  ```haskell
  data Foo a = Foo Int a
  
  instance Functor Foo where
      fmap f (Foo n a) = Foo (n+1) (f a)
  
  instance Monad Foo where
      return a = Foo 1 a
      (Foo n1 a) >>= f = case f a of Foo n2 b -> Foo (n1 + n2) b
  ```
{% endnote %}



## 工具箱

事实上，如果将Monad视作计算，那么仅仅通过提供的`return`和`(>>=)`我们就可以实现很多复杂的控制结构，比如[`Control.Monad`](http://www.haskell.org/ghc/docs/latest/html/libraries/base/Control-Monad.html)里面的各种控制函数。

```haskell
sequence :: Monad m => [m a] -> m [a]
sequence [] = return []
sequence (x:xs) = do
    v <- x
    vs <- sequence xs
    return (v:vs)

forM :: Monad m => [a] -> (a -> m b) -> m [b]
forM xs f = sequence (map f xs)

when :: (Monad m) => Bool -> m () -> m ()
when p x = if p then x else return ()

liftM :: (Monad m) -> (a -> b) -> m a -> m b
liftM f x = do
    v <- x
    return (f x)

```

事实上，Monad非常强大，应用非常广泛，而它也只是人们发明的一种结构，`Arrow`是另一个强大的结构，其功能和魔力更加强大，日后学会了再讨论。