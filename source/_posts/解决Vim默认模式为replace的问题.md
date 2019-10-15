---
title: 解决Vim默认模式为replace的问题
date: 2019-08-28 14:12:00
categories: Linux
tags:
- Linux
- Vim
---

## 现象

JumpServer 中打开 vim 默认为 replace 模式

## 原因

终端的编码设置与目标机器的编码设置不同。

## 解决方案

修改 vim 配置文件，默认在 `/etc/vimrc`，用户定义的在 `~/.vimrc`，添加：

```
set termencoding=utf-8
set fileformats=unix
set encoding=prc
```

P.S. `vimrc`中注释使用英文引号。