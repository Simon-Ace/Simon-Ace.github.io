---
type: blog
title: Linux 打印并存储 tee
date: 2021-09-27
categories: 技术
tags: Linux
cover: false
meta:
  date: false
---



<!-- more -->

如果想把屏幕输出的内存重定向到文件中：

```bash
ls -l > /tmp/ll.log
```



但是这样就不会在屏幕上输出了，如果又想在屏幕显示，又想输出到文件中，可以使用 `tee`：

```bash
ls -l | tee /tmp/ll.log
```



追加写（其中一个作用是，避免 logrotate bug，新文件会填一堆0）

```bash
ls -l | tee -a /tmp/ll.log
```

