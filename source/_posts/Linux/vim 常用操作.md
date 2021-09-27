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

#### 取消搜索后高亮

```bash
# no high light search
:nohlsearch
# 简写
:noh
```

#### 移动

```bash
w # 移至下一单词
b # 移至上一单词
```

#### 剪切、复制、粘贴、删除

```bash
dd 	# 删除当前行
5dd # 删除5行

yy 	# 复制当前行
5yy # 复制5行

p 	# 粘贴
```

#### 设置 tab 键长度

```bash
:set tabstop=4
```

#### 开启自动缩进

```bash
:set autoindent
ctrl+d 		# 停止自动缩进
```



#### 字符串替换

`:s`（substitute）命令用来查找和替换字符串。语法如下：

```
:{作用范围}s/{目标}/{替换}/{替换标志}
```

例如`:%s/foo/bar/g`会在全局范围(`%`)查找`foo`并替换为`bar`，所有出现都会被替换（`g`）。

**作用范围**

当前行：

```
:s/foo/bar/g
```

全文：

```
:%s/foo/bar/g
```

2-11行：

```
:5,12s/foo/bar/g
```

当前行`.`与接下来两行`+2`：

```
:.,+2s/foo/bar/g
```

**替换标志**

上文中命令结尾的`g`即是替换标志之一，表示全局`global`替换（即替换目标的所有出现）。 还有很多其他有用的替换标志：

空替换标志表示只替换从光标位置开始，目标的第一次出现：

```
:%s/foo/bar
```

`i`表示大小写不敏感查找，`I`表示大小写敏感：

```
:%s/foo/bar/i
# 等效于模式中的\c（不敏感）或\C（敏感）
:%s/foo\c/bar
```

`c`表示需要确认，例如全局查找`"foo"`替换为`"bar"`并且需要确认：

```
:%s/foo/bar/gc
```

回车后Vim会将光标移动到每一次`"foo"`出现的位置，并提示

```
replace with bar (y/n/a/q/l/^E/^Y)?
```

按下`y`表示替换，`n`表示不替换，`a`表示替换所有，`q`表示退出查找模式， `l`表示替换当前位置并退出。`^E`与`^Y`是光标移动快捷键

#### 多行空格

> https://cloud.tencent.com/developer/article/1638397

 **向后缩进，实则是使用`Visual Block`模式批量添加空格以达到向后缩进的效果。**

1. 按`ctrl + v`组合键进入`Visual Block`模式；
2. 按`shift + i`组合键进入编辑模式；
3. 输入需要缩进的空格数量；（这里只有一行有效，第四步做完后才全生效） 
4. 按`esc`按键完成操作。
