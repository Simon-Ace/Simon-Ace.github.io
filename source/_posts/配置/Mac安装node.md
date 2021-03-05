---
type: blog
title: Mac安装node
date: 2020-07-10
categories: 教程
tags: 教程
cover: false
meta:
  date: false
---



Mac安装及降级node版本

<!-- more -->



## 1 安装最新版Node

1. 安装HomeBrew

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

2. 安装Node

```bash
brew install node
```

3. 验证Node是否安装成功

输入下面两条指令看是否可以都输出版本号

```bash
node -v
npm -v
```



## 2 降级Node

由于开发需要或版本兼容性，需要安装低版本的Node，按下面的方式操作

1. 卸载Node

   如果你是按前面的方法安装的Node，则用下面的命令卸载

```bash
brew uninstall node
```

2. 查看可用的Node版本

```bash
brew search node
```

输出结果：

```bash
==> Formulae
libbitcoin-node      node                 node-sass            node@12            nodebrew             nodenv			llnode               node-build           node@10              node_exporter        nodeenv
```

3. 安装你需要的版本

```bash
# 这里安装v12版本
brew install node@12
```

4. 连接Node

```bash
brew link node@12
# 这一步可能会报错, 按照提示执行命令就ok了, 比如我最后执行的是brew link --overwrite --force node@12
```

5. 检查Node是否安装成功

```bash
node -v
```