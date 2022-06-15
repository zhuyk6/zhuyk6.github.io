---
title: 'Parsec: Monadic Parser Combinators'
date: 2021-07-16 05:30:06
categories:
- Languages
- Haskell
tags: [parser, Parsec]
mathjax: true
---

本文介绍Haskell的一个著名parser库`Parsec`，本文主要关注实现部分和理论，主要参考是其论文[《Parsec: Direct Style Monadic Parser Combinators For The Real World》](https://www.microsoft.com/en-us/research/publication/parsec-direct-style-monadic-parser-combinators-for-the-real-world/)。

<!--more-->

# Grammars and Parsers

## Grammars
首先要简单复习一下编译原理的知识，文法是描述语言的规则，基本可以分为以下四类：

0. Unrestricted；
1. Context sensitive grammar；
2. Context free grammar
3. Regular grammar

简单说一下区别：

0. 0类文法没有任意限制，其产生式怎么写都行（左边可以为**空**），也代表了语言最大的集合（废话，没有任何限制）；
1. 上下文相关文法，产生式左边不为空，也可以任意写，允许左边出现多个**非终结符**和**终结符**；
2. 上下文无关文法，产生式左边仅为一个**非终结符**；
3. 正则文法，线性文法，只能写作右线性（或者左线性）。

> 上述4类文法为继承关系，描述的语言集合越来越小。

特别说明一下，“**上下文**”的含义，从文法的角度，上下文相关就是允许出现下面的产生式：
```
aA -> ahello
bA -> bworld
ABC -> epsilon
```
所以这个“**上下文**”指的就是产生式左边的结构，也符合我们的语言习惯，我的理解就是如何解读一个词的含义，需要借助上下文。

而上下文无关文法就没有这个问题，因为其产生式左边只有一个**非终结符**，只需要关心产生式右边即可。大部分计算机语言都使用改造的CFG。

## Parser

Parser是个啥，简单来说就是某种运算，输入是一个字符串（或者流），输出是一个结构（比如AST），所以很容易将Parser定义为：
```haskell
type Parser a = String -> (a, String)
```
其含义就是消耗字符串`s`，得到结果`a`和剩余的字符串`s'`。

有两种构造Parser的方法：monadic和applicative。核心区别在于下面两个函数：
```haskell
(<*>) :: Applicative Parser => Parser a -> Parser (a -> b) -> Parser b
(>>=) :: Monad Parser => Parser a -> (a -> Parser b) -> Parser b
```
仅仅看类型就能可以知道，从语义上说，monadic parser可以解析上下文相关语言，因为`Parser b`本身是依赖`Parser a`的；而applicative parser则不具备这种**上下文**的依赖，所以其至多只能描述上下文无关文法CFG。

> 虽然applicative语义更弱，不过正是其语义上就不存在前后的约束，可以很好的实现并行计算，有相关的工作，虽然我还是不知道applicative parser怎么实现，以后有机会再学习。

论文中举了一个简单的例子：parse一个xml文档。我们知道这种标签语言需要前后标签匹配，对于context-free parser，需要一次独立的parse才能够保证这种匹配；而对于monadic parser，可以很轻易的实现这一点：
```haskell
xml = 
  do { name <- openTag
     ; content <- many xml
     ; endTag name
     ; return (Node name context)
     }
  <|> xmlText
    
```
没错，虽然我们parse的是一个CFG，但是用monadic parser可以将几个简单的parser很轻易的组合成一个复杂的parser，这也是parser combinator的魅力。



## Left recursion

大多数parser combinators都无法处理**左递归**，`Parsec`也不例外。如果遇到左递归，就会进入死循环。例如：

```
expr -> expr "+" factor
factor -> number | "(" expr ")"
```

这个表达式的文法，是不能直接翻译为parser combinator的，因为存在左递归。

幸运的是，所有**左递归**的文法都可以翻译为**右递归**；同时也可以定义一个名为`chainl`的combinator，可以直接计算左结合的表达式。

> 能否让parser自己判断是否存在**左递归**？文章中提到了一种名为“sharing”的技术，大概是在context里面加入一些硬件描述，然后判断当前程序的状态。
>
> 因此，所有的combinator libraries都强制使用**预测**性的parse算法，也就是LL。



## Backtracking

对于**歧义**文法，parse tree可能有多棵，parser的返回值应该是一个列表，这种情况下，由于要考虑每种情况，所以不可避免要产生**回溯**。

然而在现实中，几乎不会需要处理“歧义文法”，`parsec`对于歧义文法，只返回一种可能的parse tree。

即使处理的文法是非歧义的，有时候我们仍然需要回溯，因为parser可能需要“看得更远一些”。一个简单的例子是分析关键词`let`：

```haskell
keywordLet = do 
  {
    string "let"
    ...
  }
  <|> ...
```

很容易发现，为了区分关键词`let`和变量`letvariable`，我们不得不在parse失败后回溯，因为在处理到第4个字符之前都无法区分这二者。

大量使用backtracking可能会导致内存泄漏，核心原因就是`choice` combinator的实现，对于`p <|> q`的一种朴素的实现，当`p` parse失败，回溯并尝试`q`。当输入很大的时候就会发生内存泄漏（爆栈）。



## LL Grammar

如何限制内存泄漏，只需要限制“先前看”的深度即可。

LL文法具有“非歧义”和“无左递归”的性质，一个文法是LL(k)的，意味着最多只需要“向前看”k个字符就可以确定文法树。例如，下面就是一个LL(2)的文法：

```
S -> PQ | Q
P -> "p"
Q -> "pq"
```

如果第一个字符是`'p'`，显然无法确定是`P`还是`Q`，不过再来一个字符就可以确定了。该文法可以写作下面的代码：

```haskell
s = (p >> q) <|> q
p = char 'p'
q = char 'p' >> char 'q'
```

然而LL文法并不总是可以像这样简单的翻译成代码，例如下面的文法：

```
S -> PQ
P -> "p" | epsilon
Q -> "pq"
```

如果翻译成代码：

```haskell
s = p >> q
p = char 'p' <|> return ()
q = char 'p' >> char 'q'
```

对于输入串`"pq"`，会先parse `p`，消耗一个字符`'p'`，然后parse `q`，失败。

一般来说，对于 $PQ$ ，其中 $P \Rightarrow^\star \epsilon$ ，并且 $\text{First}(P) \bigcup \text{First}(Q) \neq \emptyset$ ，文法需要改写为 $P'Q \ |\  Q$ ， 其中 $P'$ 不再生成 $\epsilon$ 。

LL($\infty$) 是一类强大的文法，任意非歧义的CFG都可以转化为LL($\infty$)。在实践中，很多语言都需要任意的“向前看”，例如Haskell中的类型标记或C中的声明。



# Restricting lookahead

通过前面的一系列铺垫，为了解决**内存泄漏**并添加良好的**错误处理**，`parsec`限制“向前看”的行为，所以`parsec`中的parser是LL(1) parser——永远只根据当前的一个字符来决定分支。也就是说，`p <|> q`意味着，当且仅当`p` parse失败并且没有消耗任何输入，才会尝试`q`。

为了实现LL(1)，需要对输入的消耗进行跟踪，如下定义：

```haskell
newtype Parser a = Parser { unParser :: String -> Consumed a}
data Consumed a = Consumed (Reply a) | Empty (Reply a)
data Reply a = Ok a String | Error
```

该模型将parser的结果分成四类：

|         |     `Consumed`     |     `Empty`      |
| :-----: | :----------------: | :--------------: |
|  `Ok`   | 消耗符号，结果正确 | 不消耗，结果正确 |
| `Error` | 消耗符号，结果错误 | 不消耗，结果错误 |



## Basic combinators

根据上面的定义，实现一些基本的parser combinator。

首先是Monad的定义，`return`应该永远成功，并且不消耗符号。

```haskell
return x = Parser $ \s -> Empty (Ok x s)
```

`(>>=)`就复杂很多，因为parse的结果被我们分成了四类，所以这里需要分类讨论`(p >>= q)`：如果`p`错误，那就直接返回错误，如果`p`和`q`都没有错误，`Consumed`的表格如下：

|   `p`    |   `q`    | `p >>= q` |
| :------: | :------: | :-------: |
|  Empty   |  Empty   |   Empty   |
|  Empty   | Consumed | Consumed  |
| Consumed |  Empty   | Consumed  |
| Consumed | Consumed | Consumed  |

```haskell
p >>= f = Parser $ \s -> case runParser p s of
  Empty (Ok x s')    -> runParser (f x) s'
  Empty Error        -> Empty Error
  Consumed (Ok x s') -> case runParser (f x) s' of
                          Consumed reply -> Consumed reply
                          Empty reply    -> Consumed reply
  Consumed Error     -> Consumed Error
```

接下来就是`(<|>)`，一定要注意，这是LL(1)的，也就是只有`Empty Error`，才会尝试第二个parser。（论文中当`p`返回`Empty Ok`也会尝试`q`，不过最新的`parsec`已经改为仅当失败零消耗才尝试备选项）

```haskell
p <|> q = Parser $ \s -> case runParser p s of
  Empty Error -> runParser q s
  result      -> result
```

下面就可以定义一些简单的parser combinator：

```haskell
satisfy :: (Char -> Bool) -> Parser Char
satisfy test = Parser $ \s -> case s of
  []                 -> Empty Error
  (c:cs) | test c    -> Consumed (Ok c cs)
         | otherwise -> Empty Error

char c = satisfy (== c)
letter = satisfy isAlpha
digit  = satisfy isDigit

string :: String -> Parser String
string "" = return []
string (c:cs) = do
  x <- char c
  xs <- string cs
  return (x:xs)

many :: Parser a -> Parser [a]
many p =
  do { x <- p
     ; xs <- many p
     ; return (x:xs)
     }
  <|> return []

many1 p = do
  x <- p
  xs <- many p
  return (x:xs)

identifier = many1 (letter <|> digit <|> char '_')
```



## Infinite lookahead

如果只能够parse LL(1)文法，那`parsec`也太弱了，前面提到过，为了性能限制默认实现为LL(1)，但是很多时候不得不需要backtracking，也就需要“向前看”。`try`就是干这个用的。

```haskell
try :: Parser a -> Parser a
try p = Parser $ \s -> case runParser p s of
  Consumed Error -> Empty Error
  other          -> other
```

我们会发现，`try`只是在`p`失败后，将`Consumed`修改为`Empty`，重新抛出错误。也就是说，`p`的失败将不会消耗符号，所以可以将`try`和`(<|>)`组合在一起，`try p <|> q`当`p`失败，回溯并尝试`q`。

因此，如果我们将`try`放在所有地方，也就重新得到了LL($\infty$)，不过一定需要留意，因为`try`会发生回溯，所以可能会存在潜在的内存泄漏风险，实际使用中应该尽量避免使用`try`。

不过前文也说过，`try`的使用不可避免，还是`let`的例子：

```haskell
expr = 
  do {
      string "let"
    ; whiteSpace
    ; letExpr
    } 
  <|> identifier
```

这样写是错误的，因为当第一部分错误时，已经产生了字符消耗，不会跳转到第二个分支，正确的写法是下面这样：

```haskell
expr = 
  try do {
    string "let"
  ; whiteSpace 
  ; letExpr
  }
  <|> identifier
```



# Error Messages

限制LL(1)的好处，除了性能优势外，也很容易实现“报错”。

首先，我们将附加一个跟踪信息，用来标识当前所处的字符串的位置。

除了位置，我们还可以在parse的过程中，动态生成产生式的First集合，其含义就是当前“期待”的字符。

> 增加的这些辅助信息并不会导致parser增加很大的开销，因为Haskell是lazy的，只有在需要的时候（遇到错误报错）才会计算这些信息。

```haskell
newtype Parser a = P { unP :: State -> Consumed a }

data State = State {
    stateString :: String,  -- left input string
    statePos    :: Pos      -- current pos : (row, col)
}

data Consumed a =
    Consumed (Reply a)
  | Empty (Reply a)
  deriving (Functor)

data Reply a =
    Ok a State Message
  | Error Message
  deriving (Functor)

data Message = Msg Pos String [String]  -- error's pos, unexpected and expected

type Pos = (Int, Int) -- row and column 
```

## Basic parser

修改后的定义，其Monad实现也要对错误信息进行处理：

```haskell
parserReturn :: a -> Parser a
parserReturn x = P $ \s -> Empty $ Ok x s $ Msg (statePos s) [] []

parserBind :: Parser a -> (a -> Parser b) -> Parser b
parserBind m k = P $ \s -> case unP m s of
    Consumed (Ok a s' _) -> case unP (k a) s' of
                                Consumed reply -> Consumed reply
                                Empty reply    -> Consumed reply
    Empty (Ok a s' _)    -> case unP (k a) s' of
                                Consumed reply -> Consumed reply
                                Empty reply    -> Empty reply
    Consumed (Error msg) -> Consumed $ Error msg
    Empty (Error msg)    -> Empty $ Error msg

parserZero :: Parser a
parserZero = P $ \s -> Empty $ Error $ Msg (statePos s) [] []

parserPlus :: Parser a -> Parser a -> Parser a
parserPlus p q = P $ \s -> case unP p s of
    Empty (Error msg1) -> case unP q s of
                            Empty (Error msg2) -> mergeError msg1 msg2
                            Empty (Ok a s' msg2) -> mergeOk a s' msg1 msg2
                            consumed -> consumed
    result             -> result
  where
    mergeError msg1 msg2 = Empty $ Error $ merge msg1 msg2
    mergeOk a s msg1 msg2 = Empty $ Ok a s $ merge msg1 msg2
    merge (Msg pos s exp1) (Msg _ _ exp2) = Msg pos s $ exp1 ++ exp2

satisfy :: (Char -> Bool) -> Parser Char
satisfy test = P $ \(State s pos) -> case s of
    (c:cs) | test c -> let pos' = nextPos pos c
                           state' = State cs pos'
                       in pos' `seq` Consumed $ Ok c state' $ Msg pos [] []
           | otherwise -> Empty $ Error $ Msg pos [c] []
    []  -> Empty $ Error $ Msg pos "end of input" []
```

可以发现，其实主要修改的只是`(<|>)`，当`p <|> q`中的`p`失败后，要将`p`的错误信息和`q`的信息进行合并，主要是对Frist集合进行合并。

## Labels

`parsec`还提供了一种自定义错误信息的手段`label`函数，其中缀形式`(<?>)`。

`p <?> exp` 的含义：当`p`没有消耗符号，就将`exp`填充进First集合，表示当前期待的信息。

```haskell
(<?>) :: Parser a -> String -> Parser a
p <?> exp = P $ \s -> case unP p s of
    Empty (Error msg)   -> Empty $ Error $ expect msg exp
    Empty (Ok a s' msg) -> Empty $ Ok a s' $ expect msg exp
    other               -> other
  where
    expect (Msg pos s _) exp = Msg pos s [exp]
```

有了这个函数，我们可以非常方便的定制错误信息，如下：

```haskell
digit = satisfy isDigit <?> "digit"
letter = satisfy isAlpha <?> "letter"
char c = satisfy (==c) <?> show c
```



# 真实的`Parsec`

以上的所有内容，都来自论文[《Parsec: Direct Style Monadic Parser Combinators For The Real World》](https://www.microsoft.com/en-us/research/publication/parsec-direct-style-monadic-parser-combinators-for-the-real-world/)。根据论文我实现了一个玩具版的Parser。

真实的[`parsec`](https://hackage.haskell.org/package/parsec-3.1.14.0)是怎么实现的，有什么不一样的细节？主要区别如下：

- 使用CPS：从`parsec 3.0`以后，`ParsecT`就完全使用CPS重写了，主要原因是防止递归深度太深造成内存泄漏，而CPS相当于手动写作尾递归的形式，编译器会将其优化为迭代。
- 更复杂的错误信息处理：真实的报错肯定没有上文中那么简单，`Text.Parsec.Error`里面全是对于错误信息的处理函数，使得错误信息更加友好、便于阅读修改。
- 允许用户自定义State藏在context中：在一开始的monadic parser vs applicative parser一节中，提到过monadic parser理论上可以处理一切CSG，但是迄今为止文章甚至将`ParsecT`限制为LL(1)，还需要使用`try`才能拓展为LL($\infty$)，这之间的巨大鸿沟在那里呢？就在于状态，parser相当于一个state monad，迄今为止我们只是将代处理字符串和一些辅助信息当作state，其实我们可以将任意的信息藏在context中——例如，维护一个`Map String Val`，这样就可以实现带有“变量定义”的表达式计算。
- 提供丰富的parse工具：除了简单的字符parser combinator，`parsec`还提供了
  - `Text.Parsec.Expr`：专门针对表达式的parse，非常值的学习，下一片文章再来讨论；
  - `Text.Parsec.Perm`：允许parse目标序列的某个“排列”，不知道怎么实现的，也不知道有什么用。。。
  - `Text.Parsec.Token`：允许根据定义的`Language`规则，提供一套特化的parser combinator。

说起来貌似很复杂，其实我们实现的“玩具”基本已经具备全部功能，CPS可以只当作为了性能上的提升的一种实现，功能上只是没有提供用户自定义的State（其实实现起来巨简单，直接塞进State里面，提供`get`、`set`接口即可）。剩下的都是基于基本功能开发的工具，下一篇文章将讨论如何使用`Parsec`来parse表达式。
