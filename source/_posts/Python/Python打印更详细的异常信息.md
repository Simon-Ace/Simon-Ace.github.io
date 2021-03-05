---
type: blog
title: Python打印更详细的异常信息
date: 2019-10-21
categories: Python
tags: Python, 异常
cover: false
meta:
  date: false
---



打印Python异常信息的几种方式

<!-- more -->



## 1 简单的异常信息

```python
try:
    a = 1/0
except Exception as e:
    print(e)
```

打印最简单的message信息：

```
division by zero
```



## 2 更完整的信息

```python
import traceback

try:
    a = 1/0
except Exception as e:
    print('str(e):\t', e)
    print('repr(e):\t', repr(e))

    print('traceback.format_exc():\n%s' % traceback.format_exc()) #字符串
    traceback.print_exc() #执行函数
```

输出：

```
str(e):		 division by zero
repr(e):	 ZeroDivisionError('division by zero')
traceback.format_exc():
Traceback (most recent call last):
  File "/Users/ace/Play/test/异常信息.py", line 4, in <module>
    a = 1/0
ZeroDivisionError: division by zero

Traceback (most recent call last):
  File "/Users/ace/Play/test/异常信息.py", line 4, in <module>
    a = 1/0
ZeroDivisionError: division by zero
```



`traceback.format_exc()`和`traceback.print_exc()`都可以打印完整的错误信息

`traceback.format_exc()`返回值为字符串

`traceback.print_exc()`是一个执行函数，直接在控制台打印错误信息

