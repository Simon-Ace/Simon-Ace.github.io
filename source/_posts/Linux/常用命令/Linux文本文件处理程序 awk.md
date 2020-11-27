---
type: blog
title: Linux文本文件处理程序 awk
date: 2020-11-27
categories: 技术
tags: Linux
cover: false
meta:
  date: false
---



```bash

```



<!-- more -->

## 一、基础用法

`awk`是处理文本文件的一个应用程序，几乎所有 Linux 系统都自带这个程序。

它依次处理文件的每一行，并读取里面的每一个字段。对于日志、CSV 那样的每行格式相同的文本文件，`awk`可能是最方便的工具。



`awk`的基本用法就是下面的形式。

```bash
# 格式
$ awk 动作 文件名

# 示例
$ awk '{print $0}' demo.txt
```

上面示例中，`demo.txt`是`awk`所要处理的文本文件。前面单引号内部有一个大括号，里面就是每一行的处理动作`print $0`。其中，`print`是打印命令，`$0`代表当前行。因此上面命令是把每一行原样打印出来。



`awk`会根据空格和制表符，将每一行分成若干字段，依次用`$1`、`$2`、`$3`代表第一个字段、第二个字段、第三个字段等等。

```bash
$ echo 'this is a test' | awk '{print $3}'
a
```



下面，为了便于举例，我们把`/etc/passwd`文件保存成`demo.txt`。

```bash
root:x:0:0:root:/root:/usr/bin/zsh
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
```

这个文件的字段分隔符是冒号（`:`），所以要用`-F`参数指定分隔符为冒号

```bash
$ awk -F ':' '{ print $1 }' demo.txt
root
daemon
bin
sys
sync
```



## 二、变量

- `数字`：第几个字段（从1开始）
- `NF`：当前行有多少个字段，因此`$NF`就代表最后一个字段
- `NR`：表示当前处理的是第几行。
- `FILENAME`：当前文件名
- `FS`：字段分隔符，默认是空格和制表符。
- `RS`：行分隔符，用于分割每一行，默认是换行符。
- `OFS`：输出字段的分隔符，用于打印时分隔字段，默认为空格。
- `ORS`：输出记录的分隔符，用于打印时分隔记录，默认为换行符。
- `OFMT`：数字输出的格式，默认为`％.6g`。



变量`NF`表示当前行有多少个字段，因此`$NF`就代表最后一个字段；`$(NF-1)`代表倒数第二个字段。

> ```bash
> $ echo 'this is a test' | awk '{print $NF, $(NF-1)}'   # print中的逗号是以空格分隔的意思
> test a
> ```

变量`NR`表示当前处理的是第几行。

> ```bash
> $ awk -F ':' '{print NR ") " $1}' demo.txt
> 1) root
> 2) daemon
> 3) bin
> 4) sys
> 5) sync
> ```

上面代码中，`print`命令里面，如果原样输出字符，要放在双引号里面



## 三、函数
常用函数如下
- `toupper()`：字符转为大写
- `tolower()`：字符转为小写
- `length()`：返回字符串长度
- `substr()`：返回子字符串
- `sin()`：正弦
- `cos()`：余弦
- `sqrt()`：平方根
- `rand()`：随机数

`awk`内置函数的完整列表，可以查看[手册](https://www.gnu.org/software/gawk/manual/html_node/Built_002din.html#Built_002din)。

例：`toupper`

```bash
$ awk -F ':' '{ print toupper($1) }' demo.txt
ROOT
DAEMON
BIN
SYS
SYNC
```



## 四、条件

`awk`允许指定输出条件，只输出符合条件的行。

输出条件要写在动作的前面。

```bash
$ awk '条件 动作' 文件名
```

请看下面的例子。`print`命令前面是一个正则表达式，只输出包含`usr`的行。

```bash
$ awk -F ':' '/usr/ {print $1}' demo.txt
root
daemon
bin
sys
```

下面的例子只输出奇数行，以及输出第三行以后的行。

```bash
# 输出奇数行
$ awk -F ':' 'NR % 2 == 1 {print $1}' demo.txt
root
bin
sync

# 输出第三行以后的行
$ awk -F ':' 'NR >3 {print $1}' demo.txt
sys
sync
```

下面的例子输出第一个字段等于指定值的行。

```bash
$ awk -F ':' '$1 == "root" {print $1}' demo.txt
root

$ awk -F ':' '$1 == "root" || $1 == "bin" {print $1}' demo.txt
root
bin
```

## 五、if 语句

`awk`提供了`if`结构，用于编写复杂的条件。

```bash
$ awk -F ':' '{if ($1 > "m") print $1}' demo.txt
root
sys
sync
```

上面代码输出第一个字段的第一个字符大于`m`的行。

`if`结构还可以指定`else`部分。

```bash
$ awk -F ':' '{if ($1 > "m") print $1; else print "---"}' demo.txt
root
---
---
sys
sync
```

## 六、相关连接

- [An Awk tutorial by Example](https://gregable.com/2010/09/why-you-should-know-just-little-awk.html), Greg Grothaus
- [30 Examples for Awk Command in Text Processing](https://likegeeks.com/awk-command/), Mokhtar Ebrahim