---
title: Applicative Parser
tags: [parser]
mathjax: false
date: 2021-08-05 20:56:22
categories:
- Languages
- Haskell
---

本文简单介绍Applicative Parser，参考 Codewars： [Writing applicative parsers from scratch](https://www.codewars.com/kata/54f1fdb7f29358dd1f00015d/train/haskell)

<!--more-->



# Type definition

```haskell
newtype Parser a = P { runP :: String -> [(String, a)] }
```

比较容易理解，返回所有可能的parse结果，结果存在一个list里面。

实现`Functor`、`Applicative`、`Alternative`的实例：

```haskell
instance Functor Parser where
    fmap f p = P $ \s -> map (\(s', a) -> (s', f a)) $ runP p s

instance Applicative Parser where
    pure a = P $ \s -> [(s, a)]
    (P fp) <*> (P xp) = P $ \s -> [(s'', f x) | (s', f) <- fp s, (s'', x) <- xp s']

instance Alternative Parser where
    empty = P $ const []
    (P p) <|> (P q) = P $ \s -> p s ++ q s

```

# Monadic vs Applicative

之前我们讨论过`parsec`是怎么实现的，`parsec`是非常典型的monadic parser，Monad是特殊的Applicative，那么monadic parser和applicative parser有什么区别？

- 从parse结果来看，monadic parser返回确定的结果，applicative parser返回不确定的结果（零个或多个）；
- 从能力上看，`bind :: m a -> (a -> m b) -> m b`可以处理CSG上下文相关文法，而`(<*>) :: f (a -> b) -> f a -> f b`只能处理CFG上下文无关文法；
- 正是因为applicative parser只能实现CFG，所以可以在parse前对“结构”进行分析，也就是“长什么样，结果什么样”，monadic parser则不可能，因为后面的parser依赖前面的结果；
- 对`Alternative`的实现不同：monadic parser从前往后尝试，成功就返回，applicative parser返回所有可能的结果；
- 正是由于以上特性，理论上applicative parser可以更好的并行，比如`(p <|> q)`可以并行处理。
- 错误处理，monadic parser 可以很容易标记处理错误；
- 编程风格，Monad是特殊的Applicative，所以都可以使用applicative的组合子风格。

# Example

```haskell
---
predP :: (Char -> Bool) -> Parser Char
predP test = P $ \case
    (x:xs) | test x -> [(xs, x)]
    _               -> []

charP :: Char -> Parser Char 
charP c = predP (== c)

stringP :: String -> Parser String
-- stringP [] = pure []
-- stringP (x:xs) = (:) <$> charP x <*> stringP xs
stringP = foldr (\c p -> (:) <$> charP c <*> p) (pure [])

spaceP :: Parser Char 
spaceP = choice $ map charP " \t\n\r"

lexeme :: Parser a -> Parser a
lexeme p = p <* many spaceP

symbolP = lexeme . stringP

between open close p = open *> p <* close

choice :: [Parser a] -> Parser a
choice = foldr (<|>) empty

--- 

runParser :: Parser a -> String -> [a]
runParser p s = [a | (s', a) <- runP p s, null s']

runParserUnique :: Parser a -> String -> Maybe a
runParserUnique p s = case runParser p s of 
    [a] -> Just a
    _   -> Nothing 

---
```

# Parse Expression

下面尝试parse算术表达式。

```haskell
data BinOp = Add | Mul deriving (Eq, Show)

data Expr = ConstE Int
          | BinOpE BinOp Expr Expr
          | NegE   Expr
          | ZeroE  
    deriving (Show)
```



当然applicative parser也无法处理“**左递归**”，首先来看一个**无歧义文法**：

```haskell
-- | Parse arithmetic expressions, with the following grammar:
--
--     expr         ::= const | binOpExpr | neg | zero
--     const        ::= int
--     binOpExpr    ::= '(' expr ' ' binOp ' ' expr ')'
--     binOp        ::= '+' | '*'
--     neg          ::= '-' expr
--     zero         ::= 'z'
-- 

exprP = constP <|> binOpExprP <|> negP <|> zeroP

constP = lexeme $ ConstE . read <$> some (predP isDigit)

binOpExprP = between (symbolP "(") (symbolP ")") 
    $ (\a f b -> BinOpE f a b) <$> exprP <*> binOpP <*> exprP


binOpP = choice [
      Add <$ symbolP "+"
    , Mul <$ symbolP "*"
    ]

negP = fmap NegE $ symbolP "-" *> exprP

zeroP = ZeroE <$ symbolP "z"
```



我们肯定不仅仅满足于无歧义文法，applicative parser优势应该是能处理歧义文法，返回所有可能的结果。

```haskell
-- ambiguous grammar:
-- E -> E' + E | E' * E | C 
-- E' -> C + E | C * E | C 
-- C -> integer 

e = (\a f b -> BinOpE f a b) <$> e' <*> binOpP <*> e <|> c 
e' = (\a f b -> BinOpE f a b) <$> c <*> binOpP <*> e <|> c 
c = constP
```

验证`1 + 2 * 3`：

```haskell
>>> runParser (e) "1+2*3"
[BinOpE Mul (BinOpE Add (ConstE 1) (ConstE 2)) (ConstE 3),
BinOpE Add (ConstE 1) (BinOpE Mul (ConstE 2) (ConstE 3))]
```

