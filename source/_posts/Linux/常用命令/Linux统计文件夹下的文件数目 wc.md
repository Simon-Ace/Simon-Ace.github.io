---
type: blog
title: Linux统计文件数目 wc
date: 2019-10-22
categories: Linux
tags: Linux
cover: false
meta:
  date: false
---



```
$ ls -l | grep "^-" | wc -l
```

<!-- more -->



## 1 统计文件夹下的文件数目

- 统计当前目录下文件的个数（不包括目录）

```
$ ls -l | grep "^-" | wc -l
```

- 统计当前目录下文件的个数（包括子目录）

```
$ ls -lR| grep "^-" | wc -l
```

- 查看某目录下文件夹(目录)的个数（包括子目录）

```
$ ls -lR | grep "^d" | wc -l
```



**命令原理：**

- `ls -l`
  - 详细输出该文件夹下文件信息
  - `ls -lR`是列出所有文件，包括子目录
- `grep "^-"`
  - 过滤`ls`的输出信息，只保留一般文件；只保留目录是`grep "^d"`
- `wc -l`
  - 统计输出信息的行数