---
type: blog
title: Shell 脚本
date: 2020-11-19
categories: 教程
tags: shell, linux, 教程
cover: false
meta:
  date: false
---

第一行

指明脚本应使用的解释器的名字

```bash
#! /bin/bash
```



编程规范：

- 大写字母表示常量，小写字母表示变量



变量

```bash
foo="yes"		# 注意等号两边不能有空格
echo $foo

b="abc efg"	# 有空格的字符串需要用引号包围
c="hhh $b"	# 使用其他变量的值
d=$(ll hhh.txt)	# 将执行命令的结果赋值
e=$((5*7))	# 算数扩展，注意是两个括号

f=aa.txt
g=${f}1			# 使用{}限定变量名的范围
```



**流控制**

符号

```bash
[[ $str1 = $str2 ]] 	# 条件表达式部分最好使用 [[]]，更为通用，且会避免[]造成的一些符号不识别
abc=-6; ((abc<0))			# (()) 是专门为整数设计的命令，支持各种算数计算
&& || !								# 与或非，需要在[[]]、(())中使用
```

if

```bash
if [[ $x = 5 ]]; then			# 等号两边有空格，也可以用 == 代替
	echo "x=5"
elif [[ $x = 10 ]]; then
	echo "x=10"
else
	echo "no"
fi
```

while

```bash
#! /bin/bash
a=5
while [[ $a>0 ]]; do
		echo $a
		a=$(( a-1 ))
done
echo finish
```

for

```bash
for (( i=0; i<5; i++)); do
	echo $i
done

# =========================
for i in A B C D; do
	echo $i
done
```





位置参数

```bash
#! /bin/bash
echo "number of args: $#"
echo $0
echo $1
echo $2
echo $3
echo $4
```

