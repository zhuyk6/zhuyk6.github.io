---
title: Using Github Branch to Back up Hexo Blog
tags: []
mathjax: false
date: 2022-06-16 08:23:32
categories:
---

本文介绍如何使用`git branch`实现在github上备份hexo blog。

<!--more-->

# Hexo 需要的文件

- `scaffolds/` ：模板
- `source/` ：md源文件
- `themes/` ：主题（实际上空）
- `.git/` ：git信息
- `.gitignore` ：git忽略的文件
- `.github/` ：github robot，检查npm版本是否可更新
- `_config.yml` ：配置
- `_config.next.yml` ：next主题配置
- `package.json` ：npm依赖的包

# git ignore

```
# .gitignore

.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
package-lock.json
```

# git branch

在github上新建一个git branch，比如`source`，将其设置为默认分支。

git clone 下来该项目，将上述 Hexo 需要的文件放在 `source` 分支下，提交即可。

# 博客迁移

在新电脑上 git clone 该项目，然后 `npm update` 安装依赖即可。

可能随着版本更新，某些配置 `_config.next.yml` 需要修改。
