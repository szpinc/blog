---
title: gitbook安装教程
date: 2021-04-12 15:03:25
tags:
    - gitbook
categories:
    - 教程
---

最近想用gitbook写一些文章，在搭建gitbook的过程踩了一些坑，gitbook代码写的比较烂，兼容性非常差，只能用nodejs 10.*的版本
记录一下安装步骤:

## 安装`nodejs`

``` bash
brew install nodejs@10
```

## 安装gitbook

``` bash
npm install gitbook-cli -g
```

使用`gitbook init`生成`SUMMAR`和`README`