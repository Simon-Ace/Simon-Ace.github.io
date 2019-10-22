---
type: blog
title: Linux后台执行命令
date: 2019-10-22
categories: Linux
tags: Linux
cover: false
meta:
  date: false
---



当在终端工作时，可能一个持续运行的作业占住屏幕输出，或终端退出时导致命令结束。为了避免这些问题，可以将这些进程放到后台运行，且不受终端关闭的影响，可使用下面的方法：

```shell
nohup command > myout.file 2>&1 &
```

<!-- more -->

## 1 后台执行命令

### 1.1 命令`&`

在命令后面加上`& `实现后台运行（控制台关掉(退出帐户时)，作业就会**停止**运行）

```
command &
```

例：`python run.py &`



### 1.2 命令`nohup`

`nohup`命令可以在你退出帐户之后继续运行相应的进程。nohup就是不挂起的意思( no hang up)

```shell
nohup command &
```

例：`nohup run.py &`



## 2 kill进程

执行后台任务命令后，会返回一个进程号，可通过这个进程号kill掉进程。

```
kill -9 进程号
```



## 3 输出重定向

由于使用前面的命令将任务放到后台运行，因此任务的输出也不打印到屏幕上了，所以需要将输出重定向到文件中，以方便查看输出内容。

- 将输出重定向到 file（覆盖）

```
command1 > file1
```

- 将输出重定向到 file（追加）

```
command1 >> file1
```

- 将 stdout 和 stderr 合并后重定向到 file
  - 2>1代表什么，2与>结合代表错误重定向，而1则代表错误重定向到一个文件1，而不代表标准输出；换成2>&1，&与1结合就代表标准输出了，就变成错误重定向到标准输出.

```
command1 > file1 2>&1
```



**完整写法：**

```
nohup command >out.file 2>&1 &
```



## 4 其他

- nohup执行python程序时，print无法输出
  - 这是因为python的输出有缓冲，导致nohup.out并不能够马上看到输出
  - python 有个-u参数，使得python不启用缓冲
  - `nohup python -u test.py > nohup.out 2>&1 &`