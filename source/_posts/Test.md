---
title: Test
date: 2021-05-30 19:40:49
mathjax: true
tags:
---

Some test example.

<!--more-->

# Level 1

## Level 2

### Level 3

#### Level 4

#### Level 5

# Lists

1. 1
2. 2
3. 3
4. 4

- 1
- 2
- 3

1. 1
    -  1
    -  2
2. 2

# Math equation

$f(x) = \int_0^1 g(x) \text{d}x$

$$\frac{1}{1-x} = \sum_{n=0}^\infty x^n$$

# Code Blocks

{% note info %}
[Tag Plugins](https://hexo.io/docs/tag-plugins#Backtick-Code-Block)
{% endnote %}

Use three backticks to show codes.

```python This is a python code.
def fac(n):
    if n == 1:
        return 1
    else:
        return n * fac(n-1)
```

Use `codeblock` to show codes.

```
{% codeblock lang:haskell This is a Haskell code. line_number:false %}
primes = filterPrime [2..]
  where
    filterPrime :: [Integer] -> [Integer]
    filterPrime (p:xs) =
          p : filterPrime [x | x <- xs, x `mod` p /= 0]
{% endcodeblock %}
```

{% codeblock lang:haskell This is a Haskell code. line_number:false %}
primes = filterPrime [2..]
  where
    filterPrime :: [Integer] -> [Integer]
    filterPrime (p:xs) =
        p : filterPrime [x | x <- xs, x `mod` p /= 0]

(<|) infixl 5
(<|) = undefined
{% endcodeblock %}

# Tabs

{% tabs Fourth unique name %}
<!-- tab Solution 1 -->
**This is Tab 1.**
<!-- endtab -->

<!-- tab Solution 2 -->
**This is Tab 2.**
<!-- endtab -->

<!-- tab Solution 3 -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}

# Notes

{% note info %}
#### Info Header
**Welcome** to [Hexo!](https://hexo.io)
{% endnote %}


{% note warning %}
#### Warning Header
**Welcome** to [Hexo!](https://hexo.io)
{% endnote %}


{% note danger %}
#### Danger Header
**Welcome** to [Hexo!](https://hexo.io)
{% endnote %}


{% note info no-icon This is a summary %}
#### Details and summary (No icon)
Note with summary: `note info no-icon This is a summary`
{% endnote %}

# Graph

{% mermaid graph TD %}
A[Hard] -->|Text| B(Round)
B --> C{Decision}
C -->|One| D[Result 1]
C -->|Two| E[Result 2]
{% endmermaid %}

{% mermaid stateDiagram %}
[*] --> Still
Still --> [*]
Still --> Moving
Moving --> Still
Moving --> Crash
Crash --> [*]
{% endmermaid %}