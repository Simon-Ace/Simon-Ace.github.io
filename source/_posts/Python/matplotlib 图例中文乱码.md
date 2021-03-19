---
type: blog
title: matplotlib 图例中文乱码
date: 2021-03-05
categories: 教程
tags: python
cover: false
meta:
  date: false
---

<!-- more -->

1、下载中文字体（黑体，看准系统版本）

https://www.fontpalace.com/font-details/SimHei/

2、解压之后在系统当中安装好，我的是Mac，打开字体册就可以安装了，Windows的在网上搜一下吧

3、找到`matplotlib`字体文件夹，例如：`matplotlib/mpl-data/fonts/ttf`，将SimHei.ttf拷贝到ttf文件夹下面

```python
# 执行下面的找到文件夹
>>> import matplotlib
>>> print(matplotlib.matplotlib_fname())
/Users/xxx/Library/Python/3.8/lib/python/site-packages/matplotlib/mpl-data/matplotlibrc

# 即前缀就是 /Users/xxx/Library/Python/3.8/lib/python/site-packages/matplotlib
```

4、修改配置文件`matplotlibrc` （上一步看到的位置），修改下面三项配置

```bash
font.family         : sans-serif        

font.sans-serif     : SimHei, Bitstream Vera Sans, Lucida Grande, Verdana, Geneva, Lucid, Arial, Helvetica, Avant Garde, sans-serif   

axes.unicode_minus  : False #作用就是解决负号'-'显示为方块的问题
```

5、重新加载字体，在Python中运行如下代码即可：

```python
from matplotlib.font_manager import _rebuild
_rebuild() #reload一下
```

