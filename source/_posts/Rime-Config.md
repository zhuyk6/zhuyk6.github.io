---
title: Rime Config
tags: []
mathjax: false
date: 2022-06-20 19:10:44
categories:
- Tools
---

Rime configuration.

<!--more-->

# Add symbols

[github issues 373](https://github.com/rime/librime/issues/373)

Add a file `symbols.custom.yaml` .

```yaml
# symbols.custom.yaml
patch:
  punctuator/symbols/+:
    '/ok': [👌,👌😁, 👌🤓]
```

Add greek symbols:

```yaml
# symbols.custom.yaml
patch:
  punctuator/symbols/+:
    "/alpha": ["Α", "α"]
    "/beta": ["Β", "β"]
    "/gamma": ["Γ", "γ"]
    "/delta": ["Δ", "δ"]
    "/epsilon": ["Ε", "ε"]
    "/zeta": ["Ζ", "ζ"]
    "/eta": ["Η", "η"]
    "/theta": ["Θ", "θ"]
    "/iota": ["Ι", "ι"]
    "/kappa": ["Κ", "κ"]
    "/lambda": ["Λ", "λ"]
    "/mu": ["Μ", "μ"]
    "/nu": ["Ν", "ν"]
    "/xi": ["Ξ", "ξ"]
    "/omicron": ["Ο", "ο"]
    "/pi": ["Π", "π"]
    "/rho": ["Ρ","ρ"]
    "/sigma": ["Σ", "σ", "ς"]
    "/tau": ["Τ", "τ"]
    "/upsilon": ["Υ", "υ"]
    "/phi": ["Φ", "φ"]
    "/chi": ["Χ", "χ"]
    "/psi": ["Ψ", "ψ"]
    "/omega": ["Ω", "ω"]
```
