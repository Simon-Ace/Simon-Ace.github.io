---
type: blog
title: Shell 编程
date: 2020-11-19
categories: 教程
tags: shell, linux, 教程
cover: false
meta:
  date: false
---

## 0 基础

第一行

指明脚本应使用的解释器的名字

```bash
#! /bin/bash
```

编程规范：

- 大写字母表示常量，小写字母表示变量



## 1 变量

**变量**

```bash
# 注意等号两边不能有空格
foo="yes"
echo $foo

# 有空格的字符串需要用引号包围
b="abc efg"	

# 使用其他变量的值
c="hhh $b"

# 将执行命令的结果赋值
d=$(ls -la)	
d1=`ls -la` # ``等价 $()

# 算数扩展，注意是两个括号
e=$((5*7))	

# 使用{}限定变量名的范围
f=aa.txt
g=${f}1			
```

**环境变量**

```bash
export xx=xxx		# 设置环境变量
source xxx_file	# 让文件中的环境变量立即生效
echo $xx				# 输出变量的值
```

**位置参数变量**

```bash
$n		# $0为命令本身；$1-$9代表第一到第九个参数；${10}十以上的用大括号括起来
$*		# 代表命令行中所有参数，并把所有参数看成一个整体
$@		# 代表命令行中所有参数，但把每个参数区分对待？
$#		# 参数个数
```

**预定义变量**

```bash
$$		# 当前进程号
$!		# 后台运行的最后一个进程的进程号
$?		# 最后一次执行命令的返回状态，0代表成功，其他都是失败 可自定义
```



## 2 条件判断

**运算符**

```bash
$((m+n))			# $(()) 中间写运算式
$[m+n]				# 推荐这种方式
expr m + n		# 不推荐
```

**条件判断**

test

```bash
# 写法一
test expression

# 写法二
[ expression ]

# 写法三
[[ expression ]]		# 推荐使用这种写法，包含前两种的用法，且还支持模式匹配

# 数字判断
(( expression ))
```

字符串判断

```bash
[ -n string ]				# 如果字符串string的长度大于零，则为真
[ -z string ]				# 如果字符串string的长度为零，则为真
[ string1 = string2 ]		# 如果string1和string2相同，则为真
[ string1 == string2 ] 	# 等同于[ string1 = string2 ]
[ string1 != string2 ]	# 如果string1和string2不相同，则为真
[ string1 '>' string2 ]	# 如果按照字典顺序string1排列在string2之后，则为真
[ string1 '<' string2 ]	# 如果按照字典顺序string1排列在string2之前，则为真
```

整数判断

```bash
(( m > n ))		# 可以直接用 > < == >= <= !=，空格都要有！
```

逻辑判断

```bash
[[ expr1 && expr2 ]]	# && || !
```



## 3 流控制

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

case

```bash
case $变量名 in
"值 1"）
如果变量的值等于值 1，则执行程序 1
;;

"值 2"）
如果变量的值等于值 2，则执行程序 2
;;

*）
如果变量的值都不是以上的值，则执行此程序
;;

esac
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
for (( i=0; i<5; i++ )); do
	echo $i
done

# =========================
for i in A B C D; do
	echo $i
done
```





