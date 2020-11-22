---
type: blog
title: console 日志输出到文件
date: 2020-10-09
categories: 教程
tags: 教程
cover: false
meta:
  date: false
---



```bash
some_command 2>&1 | tee output.txt
```



<!-- more -->

- 在Linux中，如果想将一个程序在控制台中的输出字符输出到文件中，不保留控制台内的文字，可以用下面命令：

```bash
some_command > output.txt
```

- 命令结果会输出到`output.txt`中，换成`>>`可以追加到文件末尾

- 但如果想输出到文件同时，保留控制台的内容，需要使用tee命令，示例如下：

```bash
some_command | tee output.txt
```

- 有时会发现上述命令后屏幕有输出，但文件内容为空，此时可能是由于some_command输出的字符从std error文件描述符输出，需要先将std error的输出导向到std output：

```bash
some_command 2>&1 | tee output.txt
```

其中，2代表std error，1代表std output，>&是linux中fd到fd的重定向操作符。

