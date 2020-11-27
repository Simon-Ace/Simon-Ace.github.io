---
type: blog
title: Linux常用命令
date: 2020-11-27
categories: 技术
tags: Linux, 压缩
cover: false
meta:
  date: false
---

## 一、基础指令

- xargs

[Linux输出转换命令 xargs](./常用命令/Linux输出转换命令 xargs.md)

`xargs`命令的作用，是将标准输入转为命令行参数。

原因：大多数命令都不接受标准输入作为参数，只能直接在命令行输入参数，这导致无法用管道命令传递参数

```bash
$ echo "hello world" | xargs echo
hello world
```







## 二、指令组合

**输出中间结果**

使用 tee 指令，将中间结果进行输出

```bash
find ./ -i "*.java" | tee JavaList | grep Spring
```

