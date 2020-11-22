---
type: blog
title: vim 常用操作
date: 2020-09-17
categories: 教程
tags: vim, 教程
cover: false
meta:
  date: false
---



```bash
# 取消高亮
:noh
```



<!-- more -->

取消搜索后高亮

```bash
# no high light search
:nohlsearch
# 简写
:noh
```

移动

```bash
w # 移至下一单词
b # 移至上一单词
```

剪切、复制、粘贴、删除

```bash
dd 	# 删除当前行
5dd # 删除5行

yy 	# 复制当前行
5yy # 复制5行

p 	# 粘贴
```

设置 tab 键长度

```bash
:set tabstop=4
```

开启自动缩进

```bash
:set autoindent
ctrl+d 		# 停止自动缩进
```

