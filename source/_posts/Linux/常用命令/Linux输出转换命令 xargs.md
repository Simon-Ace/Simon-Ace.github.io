---
type: blog
title: Linux输出转换命令 xargs
date: 2020-11-27
categories: 技术
tags: Linux, 压缩
cover: false
meta:
  date: false
---



```bash
$ echo "hello world" | xargs echo
```



<!-- more -->

## 一、基本用法

`xargs`命令的作用，是将标准输入转为命令行参数。

原因：大多数命令都不接受标准输入作为参数，只能直接在命令行输入参数，这导致无法用管道命令传递参数

如下面 echo 不接受标准输出做参数，可用 xargs 做转换：

```bash
$ echo "hello world" | xargs echo
hello world
```



## 二、参数

### `-d` 指定分隔符

默认情况下，`xargs`将换行符和空格作为分隔符，把标准输入分解成一个个命令行参数。

```bash
$ echo "one two three" | xargs mkdir
```

上面代码中，`mkdir`会新建三个子目录，执行`mkdir one two three`。

`-d`参数可以更改分隔符

```bash
$ echo -e "a\tb\tc" | xargs -d "\t" echo
a b c
```

上面的命令指定制表符`\t`作为分隔符，所以`a\tb\tc`就转换成了三个命令行参数。`echo`命令的`-e`参数表示解释转义字符。



### `-p -t`打印将要执行的命令

`-p`参数打印出要执行的命令，询问用户是否要执行。

```bash
$ echo 'one two three' | xargs -p touch
touch one two three ?...
```

`-t`参数则是打印出最终要执行的命令，然后直接执行，不需要用户确认。

```bash
$ echo 'one two three' | xargs -t rm
rm one two three
```



### `-I` 传递参数起别名

如果`xargs`要将命令行参数传给多个命令，可以使用`-I`参数。【貌似，会按空格或回车对参数进行分割，然后重复执行命令，而不是当成命令的多个参数】

`-I`指定每一项命令行参数的替代字符串。

```bash
$ cat foo.txt
one
two
three

$ cat foo.txt | xargs -I file sh -c 'echo file; mkdir file'
one 
two
three

$ ls 
one two three
```

上面代码中，`foo.txt`是一个三行的文本文件。我们希望对每一项命令行参数，执行两个命令（`echo`和`mkdir`），使用`-I file`表示`file`是命令行参数的替代字符串。执行命令时，具体的参数会替代掉`echo file; mkdir file`里面的两个`file`。



### `-l -L` 指定多少行作为一个命令行参数

```bash
$ echo -e "a\nb\nc" | xargs -L 1 echo
a
b
c
```



### `-n` 指定一行内多项作为一个命令行参数

```bash
$ echo {0..9} | xargs -n 2 echo
0 1
2 3
4 5
6 7
8 9
```



### `--max-procs` 多线程执行

`xargs`默认只用一个进程执行命令。如果命令要执行多次，必须等上一次执行完，才能执行下一次。

`--max-procs`参数指定同时用多少个进程并行执行命令。`--max-procs 2`表示同时最多使用两个进程，`--max-procs 0`表示不限制进程数。

```bash
$ docker ps -q | xargs -n 1 --max-procs 0 docker kill
```

上面命令表示，同时关闭尽可能多的 Docker 容器，这样运行速度会快很多

