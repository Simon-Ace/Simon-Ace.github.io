---
type: blog
title: Hexo多台电脑同步
date: 2019-10-16
categories: 教程
tags: Hexo
cover: false
meta:
  date: false
---

如果换了电脑该如何同步Hexo的源文件？把hexo文件从一个电脑cope到另外一个电脑吗？答案肯定不是这样的，因为这里面有好多依赖包，好几万个文件呢，这样显然不合理。

本文提供一种多台电脑同步源文件的方法。

<!-- more -->

## 0 解决思路

使用GitHub的分支！在博客对应的仓库中新建一个分支。一个分支用来存放Hexo生成的网站原始的文件，另一个分支用来存放生成的静态网页。



## 1 创建分支

### 1.1 创建新分支

命令行操作：

GitHub操作：

点击branch按钮，输入新的分支名`source`，点创建。

### 1.2 设置默认分支

准备在`source`分支中存放源文件，`master`中存放生成的网页，因此将`source`设置为默认分支，方便同步文件。

在仓库`->Settings->Branches->Default branch`中将默认分支设为`source`，save保存



## 2 源文件上传到GitHub

1. 选好一个本地文件夹，执行

`git clone git@github.com:Simon-Ace/Simon-Ace.github.io.git(替换成你的仓库)`

2. 在克隆到本地的`Simon-Ace.github.io`中，把除了.git 文件夹外的所有文件都删掉

3. 把之前我们写的博客源文件全部复制过来，除了`.deploy_git`

复制过来的源文件应该有一个`.gitignore`，用来忽略一些不需要的文件，如果没有的话，自己新建一个，在里面写上如下，表示这些类型文件不需要git：

```shell
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
```

注意，如果你之前克隆过theme中的主题文件，那么应该把主题文件中的`.git`文件夹删掉，因为git不能嵌套上传。

4. 提交更改

```shell
git add .
git commit –m "add branch"
git push 
```



---

参考文章：

https://juejin.im/post/5acf22e6f265da23994eeac9

https://www.zhihu.com/question/21193762