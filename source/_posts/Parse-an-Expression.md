---
title: Parse an Expression
date: 2021-07-20 07:16:40
categories:
- Languages
- Haskell
tags: [parser, expression]
mathjax: false
---

这篇博客讨论如何使用Haskell的`Parsec`库来实现对表达式的parse，本文主要参考[《Intro to Parsing with Parsec in Haskell》](https://jakewheat.github.io/intro_to_parsing/#_simple_expr)。

<!--more-->

# 使用`Parsec`

首先介绍一些简单的需要使用的parser combinators。

```haskell
satisfy :: Stream s m Char => (Char -> Bool) -> ParsecT s u m Char
```

```
*Main> parseTest (satisfy isLower ) "abc"
'a'
*Main> parseTest (satisfy isLower ) "Abc"
parse error at (line 1, column 1):
unexpected "A"
```

这是一个非常基本的parser，很多parser都可以通过`satisfy`构造出来，只需要调整`test`函数即可。

```haskell
char c = satisfy (==c)

digit = satisfy isDigit

letter = satisfy isLetter

anyChar = satisfy (const True)

oneOf cs = satisfy (`elem` cs)

noneOf cs = satisfy (not . (`elem` cs))

space = oneOf [' ', '\t', '\n']
```

下面介绍一下`many`，`many p`相当于正则表达式中的`p*`，匹配零次或多次`p`，将结果返回一个列表；`many1`是匹配一次或多次。`many`的一种可能的实现如下：

```haskell
many p = 
  do {
    x <- p; 
    xs <- many p; 
    return (x:xs)
  }
  <|> return []

many1 p = do
  x <- p
  xs <- many p
  return (x:xs)
```

有了`many`就可以实现一些字面值的parse，比如`integer`、`identifier`：

```haskell
integer :: Parsec String u Int
integer = read <$> many1 digit

identifier :: Parsec String u String
identifier = do
  fc <- letter
  cs <- many $ choice [letter, digit, char '_']
  return (fc:cs)
```

```
*Main> parseTest integer "12341"
12341
*Main> parseTest identifier "xs1xA"
"xs1xA"
```

下面说明如何匹配括号：

```haskell
parens :: Parsec String u a -> Parsec String u a
-- parens p = char '(' *> p <* char ')'
parens p = do
  char '('
  r <- p
  char ')'
  return r
```

很容易发现，这样子是不允许中间有空白的。
```
*Main> parseTest (parens integer) "(123)"
123
*Main> parseTest (parens integer) "( 123 )"
parse error at (line 1, column 2):
unexpected " "
expecting digit
```

一种简单的改进，在`char '('`后面添加`spaces`。但是这样手动添加空白太过于繁琐，于是就有了lexeme parsing——每个符号parser应该同时消耗并忽略尾部的空白。

{% note warning %}
lexeme parser自动消耗末尾的空白，一定不要忘记开头的空白，只需要将最外层的parser `p`修改为`(spaces *> p)`。
{% endnote %}

```haskell
type Parser = Parsec String ()

lexeme :: Parser a -> Parser a
lexeme p = p <* spaces

symbol = lexeme . char

integer :: Parser Int
integer = lexeme $ read <$> many1 digit

identifier = lexeme $ do
  fc <- letter
  cs <- many $ choice [letter, digit, char '_']
  return (fc:cs)

parens p = symbol '(' *> p <* symbol ')'

parensP = spaces *> choice
  [ parens parensP >> (parens parensP <|> return ())
  , return ()
  ]
```

其中`parensP`是一个只匹配括号的parser，其产生式如下:
```
P -> '(' P ')' | epsilon | PP
```
因为这个产生式包含左递归，同时可产生空字符串，所以`parensP`才长这个样子。

```
*Main> parseTest parensP "(())("
parse error at (line 1, column 6):
unexpected end of input
expecting white space, "(" or ")"
*Main> parseTest parensP "(())()"
()
```

# Parse an Expression

首先给出Expression的产生式和AST：

```
E -> number | var | E + E | E - E | E * E | (E)
```

```haskell
data Expr = 
    Num Int
  | Var String
  | Add Expr Expr
  | Sub Expr Expr
  | Mul Expr Expr
```

这是一个简单的四则运算的产生式，很明显不能直接写作parser combinator，因为这玩意儿一看就问题多多：

- 左递归
- 不是LL(1)

再次强调，`Parsec`最擅长的是LL(1)的parsing，如果我非要直接写行不行？使用**“回溯”**是可以处理非LL(1)的情况。

左递归必须干掉，不干掉的话会发生什么？

```haskell v1 left recursion
num = Num <$> integer
var = Var <$> identifier

add = do
  x <- expr
  symbol '+'
  y <- expr
  return $ Add x y

expr :: Parser Expr
expr = choice
  [ try add
  , num
  , var
  , parens expr]
```

这里，我们使用`try`先判断是不是加法，如果不是加法，再判断后面的情况，这是因为`add`和`num`、`var`不是LL(1)的。然而这个实现是错误的，因为存在**左递归**，程序会一直在`expr`和`add`之间反复横跳。

为了“消除左递归”，我们引入一个新的变量`term`，表示加法的“**项**”，`term`应该是`expr`的终结符（First集合）组成，要包含`expr`的所有First集合的可能性。

```haskell v2 using term to eliminate left recursion
num = Num <$> integer
var = Var <$> identifier

term = num <|> var <|> parens expr

add = do
  x <- term
  symbol '+'
  y <- expr
  return $ Add x y

expr :: Parser Expr
expr = try add <|> term
```

这个实现其实已经可以满足需求了，但是还有令人不满意的地方：

- 首先这个结构是“**右结合**”的，`1+2+3`被解释成`1+(2+3)`而不是`(1+2)+3`
- 其次这个实现使用了`try`，会产生回溯，这令人不满。

```
*Main> parseTest expr "1+2"
Add (Num 1) (Num 2)
*Main> parseTest expr "1+2+3"
Add (Num 1) (Add (Num 2) (Num 3))
```

能否搞成左结合呢？类比`foldl`和`foldr`的区别，我们可以这么实现：

```haskell v3 using chainl to get left association
num = Num <$> integer
var = Var <$> identifier

term = num <|> var <|> parens expr

expr :: Parser Expr
expr = do
    e <- term
    maybeAddSuffix e
  where
    addSuffix e1 = do
        symbol '+'
        e2 <- term
        maybeAddSuffix (Add e1 e2)
    maybeAddSuffix e = addSuffix e <|> return e
```

我们可以看到，已经可以正确的得到左结合的结构。
```
*Main> parseTest expr "1+2+3"
Add (Add (Num 1) (Num 2)) (Num 3)
```

同时注意到，我们没有使用`try`，也就是说这是LL(1)的。

为什么这是正确的呢？其实思路很简单，之前我们的v2版本试图从一开始就对`expr`进行分类——你是`add`还是`term`。然而这并不合理，一个表达式的核心在于`op`，所以我们应该在`op`这个地方进行分类，这样就是LL(1)了，不需要回溯。

为了正确的产生“左结构”，我们使用了类似`foldl`的方法，搞一个`acc`，带着`acc`向后扫描即可。（其实这就是`chainl1`的实现）

前文提到，这种方法是在`op`处进行分支判断，所以只需要对`op`添加更多parser，就可以支持多种运算符。

```haskell v4 multiple operators
operator = choice [
    symbol '+' >> return Add
  , symbol '-' >> return Sub
  , symbol '*' >> return Mul
  , symbol '/' >> return Div
  ]

expr :: Parser Expr
expr = do
    e <- term
    maybeAddSuffix e
  where
    addSuffix e1 = do
        op <- operator
        e2 <- term
        maybeAddSuffix (e1 `op` e2)
    maybeAddSuffix e = addSuffix e <|> return e
```

然而这个v4版本揭示了一个很有意义的bug，如下：

```
*Main> parseTest expr "1 + 2 * 3"
Mul (Add (Num 1) (Num 2)) (Num 3)
*Main> parseTest expr "1 * 2 + 3"
Add (Mul (Num 1) (Num 2)) (Num 3)
```

没错，这是一个“**歧义文法**”。其实我们的实现并没有错，“*错的不是我，是这个世界*”，只是因为该文法本身就是有歧义的。

怎么消除歧义？数学上我们知道要定义“**优先级**”，那优先级在这里应该指什么呢？

一种直觉的思路，优先级就是一遍一遍扫，外层的优先级低，内层的优先级高（优先级高的先结合）。这里其实并不是指多次遍历，更准确的说法是：加法的“**项**”可以是一次乘法的结果，所以可以分层次地改写文法：

```
Expr -> Term | Expr '+' Term | Expr '-' Term
Term -> Factor | Term '*' Factor | Term '/' Factor
Factor -> number | variable | '(' Expr ')'
```

这就是改写后的文法，这个文法就是无歧义的，我们只需要将这个文法按照上文的方法实现即可。

```haskell v5 using chainl to realize expression with priority
expr = term `chainl1` addop
term = factor `chainl1` mulop
factor = num <|> var <|> parens expr

addop = do { symbol '+'; return Add}
    <|> do { symbol '-'; return Sub}

mulop = do { symbol '*'; return Mul}
    <|> do { symbol '/'; return Div}
```

这基本就是终极状态了，可以看到并不复杂，反而非常简洁，非常接近产生式，基本相当于直接翻译产生式，`chainl`就具备这种将“左递归”的产生式正确翻译的能力，其实现就是v4版本里面写的那样，也不复杂。

parsing expression作为常见的parser，`Text.Parsec.Expr`提供了一个非常简单的接口，基本相当于把我们的v5进行很复杂的封装，能够支持：

- 前缀单目运算
- 后缀单目运算
- 中缀双目运算

```haskell v6 using buildExpressionParser
pteTable = [[Infix (Mul <$ symbol '*') AssocLeft, Infix (Div <$ symbol '/') AssocLeft]
          , [Infix (Add <$ symbol '+') AssocLeft, Infix (Sub <$ symbol '-') AssocLeft]]

pteTerm = num <|> var <|> parens pteExpr

pteExpr = buildExpressionParser pteTable pteTerm
```

{% note warning %}
迄今为止我们的程序正确性，都基于`op`的parsing是LL(1)的，对于`>`和`>=`这样的运算符，显然就不能够区分，还是要使用`try`，在什么地方添加`try`又是一个问题。

所以对于复杂的表达式，先pass一遍生成`token`是很好的办法，然后对于`[token]`再去写parser。
{% endnote %}