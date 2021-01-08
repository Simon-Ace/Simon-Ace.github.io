---
type: blog
title: Linux文本编辑命令 sed
date: 2020-12-16
categories: 技术
tags: Linux, sed
cover: false
meta:
  date: false
---





## 一、基本介绍



## 二、常用操作

### 2.1 替换

用`s`命令替换

```bash
sed "s/替换前字符/替换后字符/" xxx.txt  # 仅替换每行第一个匹配的字符
```

```bash
➜ cat NewFile.txt
hello tom
1234
hhh                                                                                                                                                                                     ➜ sed "s/h/p/" NewFile.txt
pello tom
1234
phh
```

可以在后面添加`g`，作用于行内所有匹配的字符

```bash
➜ sed 's/h/p/g' NewFile.txt
pello tom
1234
ppp
```

**输出到文件**

```bash
# 输出到新文件
sed 's/h/p/g' NewFile.txt > nn.txt

# 原地替换（-i）
sed -i 's/h/p/g' NewFile.txt
```

### 2.2 其他

`d`命令：删除匹配行

```
sed '2,$d' my.txt
```

`p`命令：打印命令，类似grep功能

```
sed -n '1,/fish/p' my.txt
```



## 三、正则匹配

### 3.1 单个匹配

在每一行最前面(^)加点东西：

```
sed "s/^/#/g" pets.txt
```

在每一行最后面($)加点东西：

```
sed "s/$/ --- /g" pets.txt
```

正则表达式的一些最基本的东西：

- `^` 表示一行的开头。如：`/^#/` 以`#`开头的匹配。
- `$` 表示一行的结尾。如：`/}$/` 以`}`结尾的匹配。
- `\<` 表示词首。 如 `\<abc` 表示以 abc 为首的詞。
- `\>` 表示词尾。 如 `abc\>` 表示以 abc 結尾的詞。
- `.` 表示任何单个字符。
- `*`表示某个字符出现了0次或多次。
- `[]` 空格或字符集合。 如：`[abc]`表示匹配a或b或c，还有`[a-zA-Z]`表示匹配所有的26个字符。如果其中有`^`表示反，如`[^a]`表示非a的字符。

正规则表达式是一些很牛的事，比如我们要去掉某html中的tags：

如果你这样搞的话，就会有问题:

```
sed "s/<.*>//g" html.txt
```

要解决上面的那个问题，就得像下面这样，其中的"[^>]"指定了除了>的字符重复0次或多次。

```
sed "s/<[^>]*>//g" html.txt
```

我们再来看看指定需要替换第3行的内容：

```
sed "3s/my/your/g" pets.txt
```

下面的命令只替换第3到第6行的文本：

```
sed "3,6s/my/your/g" pets.txt
```

只替换每一行的第一个s：

```
sed "s/s/S/1" my.txt
```

只替换每一行的第二个s：

```
sed "s/s/S/2" my.txt
```

只替换第一行的第3个以后的s：

```
sed "s/s/S/3g" my.txt
```

### 3.2 多个匹配

如果我们需要一次替换多个模式，可参看下面的示例：（第一个模式把第一行到第三行的my替换成your，第二个则把第3行以后的This替换成了That）。

```
sed "1,3s/my/your/g; 3,$s/This/That/g" my.txt
```

上面的命令等价于：（注：下面使用的是sed的-e命令行参数）

```
sed -e "1,3s/my/your/g" -e "3,$s/This/That/g" my.txt
```

我们可以使用&来当做被匹配的变量，然后可以在基本左右加点东西。如下所示：

```
sed "s/my/[&]/g" my.txt
```

### 3.3 圆括号匹配

使用圆括号匹配的示例：（圆括号括起来的正则表达式所匹配的字符串会可以当成变量来使用，sed中使用的是\1,\2…）

```
sed "s/This is my \([^,]*\),.*is \(.*\)/\1:\2/g" my.txt
```

上面这个例子中的正则表达式有点复杂，解开如下（去掉转义字符）：

正则为：This is my ([^,]*),.*is (.*)

匹配为：This is my (cat),……….is (betty)

然后：\1就是cat，\2就是betty



---

### 相关链接

https://www.cnblogs.com/ggjucheng/archive/2013/01/13/2856901.html

https://man.linuxde.net/sed

https://coolshell.cn/articles/9104.html

https://zhuanlan.zhihu.com/p/145661854

https://www.cnblogs.com/along21/p/10366886.html