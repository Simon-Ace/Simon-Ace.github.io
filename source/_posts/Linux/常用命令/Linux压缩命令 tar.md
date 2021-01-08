---
type: blog
title: Linux压缩命令 tar
date: 2020-09-01
categories: 技术
tags: Linux, 压缩
cover: false
meta:
  date: false
---



```bash
# 解压 tar包
tar -xvf file.tar 

# 解压tar.gz
tar -xzvf file.tar.gz 

# 1.15版本后 tar 自动识别压缩方式
tar -xvf filename.tar.gz
```



<!-- more -->

## 一、常用压缩参数 

**必选参数，压缩解压都要用到其中一个：**

-c:  建立压缩档案

-x：解压

-t：查看内容

-r：向压缩归档文件末尾追加文件

-u：更新原压缩包中的文件

**可选参数：**

-z：有gzip属性的

-j： 有bz2属性的

-Z：有compress属性的

-v：显示所有过程

-O：将文件解开到标准输出

**下面的参数-f是必须的**

-f: 使用档案名字，切记，这个参数是最后一个参数，后面只能接档案名。



## 二、举个栗子

**压缩**

```bash
# 将目录里所有jpg文件打包成tar.jpg
tar -cvf jpg.tar *.jpg 

# 将目录里所有jpg文件打包成jpg.tar后，并且将其用gzip压缩，生成一个gzip压缩过的包，命名为jpg.tar.gz
tar -czf jpg.tar.gz *.jpg  

# rar格式的压缩，需要先下载rar for linux
rar a jpg.rar *.jpg 

# zip格式的压缩，需要先下载zip for linux
zip jpg.zip *.jpg
# zip 文件夹
zip -r folder.zip folder
```

**解压**

```bash
# 解压 tar包
tar -xvf file.tar 

# 解压tar.gz
tar -xzvf file.tar.gz 

# 解压 tar.bz2
tar -xjvf file.tar.bz2  

# 解压rar
unrar e file.rar 

# 解压zip
unzip file.zip 
```

从1.15版本开始tar就可以自动识别压缩的格式,故不需人为区分压缩格式就能正确解压

```bash
tar -xvf filename.tar.gz
tar -xvf filename.tar.bz2
tar -xvf filename.tar.xz
tar -xvf filename.tar.Z
```





