---
type: blog
title: Python-加快pip安装速度
date: 2019-10-16
categories: 教程
tags: Hexo
cover: false
meta:
  date: false
---

 

PIP安装时使用国内镜像，加快下载速度




<!-- more -->



## 0 国内源

清华：https://pypi.tuna.tsinghua.edu.cn/simple

阿里云：http://mirrors.aliyun.com/pypi/simple/

中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/

华中理工大学：http://pypi.hustunique.com/

山东理工大学：http://pypi.sdutlinux.org/ 

豆瓣：http://pypi.douban.com/simple/



## 1 临时使用

 可以在使用pip的时候加参数`-i https://pypi.tuna.tsinghua.edu.cn/simple`

例如：

` pip install -i https://pypi.tuna.tsinghua.edu.cn/simple numpy`



## 2 永久修改

这样就不用每次都添加国内镜像源地址了

Linux下，修改` ~/.pip/pip.conf `（没有就创建一个文件夹及文件）

打开文件，添加内容：

```python
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host=mirrors.aliyun.com
```

 windows下，直接在user目录中创建一个pip目录，如：C:\Users\xx\pip，新建文件pip.ini，

内容同上