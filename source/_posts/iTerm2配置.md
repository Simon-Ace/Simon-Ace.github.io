---
type: blog
title: iTerm2配置
date: 2020-07-10
categories: 教程
tags: 教程
cover: false
meta:
  date: false
---



iTerm2配置

Oh-my-zsh安装，主题配置

<!-- more -->



## 1 安装iTerm2

iTerm2 是一款完全免费的，专为 Mac OS 用户打造的命令行应用。直接在官网上 http://iterm2.com/ 下载并安装即可。

设置为默认终端

<img src="https://raw.githubusercontent.com/shuopic/ImgBed/master/NoteImgs/Snipaste_2020-07-10_14-26-44.jpg" alt="Snipaste_2020-07-10_14-26-44" style="zoom: 33%;" />

## 2 安装 oh-my-zsh

bash是mac中terminal自带的shell，把它换成oh-my-zsh，这个的功能要多得多。拥有语法高亮，命令行tab补全，自动提示符，显示Git仓库状态等功能。

```bash
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

**解决权限问题**

如果安装完重启iterm之后，出现下面的提示：

```
[oh-my-zsh] Insecure completion-dependent directories detected:
drwxrwxrwx 7 hans admin 238 2 9 10:13 /usr/local/share/zsh
drwxrwxrwx 6 hans admin 204 10 1 2017 /usr/local/share/zsh/site-functions

......
```

解决方法：

```bash
chmod 755 /usr/local/share/zsh
chmod 755 /usr/local/share/zsh/site-functions
```



## 3 配置主题



## 4 Vim配置

设置鼠标滚动

![image-20200710182315449](https://raw.githubusercontent.com/shuopic/ImgBed/master/NoteImgs/image-20200710182315449.png)