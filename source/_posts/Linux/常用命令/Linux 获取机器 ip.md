---
type: blog
title: Linux 获取机器 ip
date: 2021-08-17
categories: 技术
tags: Linux
cover: false
meta:
  date: false
---



```bash
# 推荐
hostname -I
```



<!-- more -->

## 方法一：利用 `ifconfig` 命令获取

最熟悉的看 ip 方法，但会打印出一大堆信息，需要筛选

可以使用如下的命令

```bash
ifconfig -a | grep -o -e 'inet [0-9]\{1,3\}.[0-9]\{1,3\}.[0-9]\{1,3\}.[0-9]\{1,3\}' | grep -v "127.0.0" | awk '{print $2}'
```

命令解释：

1）grep 命令可以用于从文件或标准输入中查找指定的字符串。

`-o` 表示“仅显示匹配的内容”。

`-e` 表示“支持正则表达式查找” 。

`-v` 表示取反，查找不包含相应字符串的行。

2）正则表达式部分

`[0-9]\{1,3\}`，其中`[0-9]`表示匹配“0-9”中的任意一个数字，`{1,3}`表示“前面的匹配数字的个数是1个至3个之间”。

3）awk

`awk '{print $2}'` 表示按空格分隔，取第二个



## 方法二：hostname -i

命令简单，但它的缺点在man帮助文档中被明文指出：它仅能在主机名能被解释时方便正常工作。

`hostname -i`



## 方法三：hostname -I

与方法二一样命令简单，能够显示所有配置网口的ip地址。同时它不存在方法二的缺点，即便当前主机名不能被解释时，也能正常工作。

在脚本中获取IP推荐使用该方法。

`hostname -I`

