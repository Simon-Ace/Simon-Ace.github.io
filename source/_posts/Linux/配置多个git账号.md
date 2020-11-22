---
type: blog
title: 一台电脑配置多个git账号
date: 2019-10-14
categories: 教程
tags: git
cover: false
meta:
  date: false
---
## 1 清除git全局设置

如果配置第一个账号的时候使用`git config --global`设置过，就先要取消掉，否则两个账号肯定会冲突

```shell
# 取消global
git config --global --unset user.name
git config --global --unset user.email
```

<!-- more -->

## 2 生成新账号的SSH keys

### 2.1 用 `ssh-keygen` 命令生成密钥

```shell
$ ssh-keygen -t rsa -C "new email"
```

平时都是直接回车，默认生成 `id_rsa` 和 `id_rsa.pub`。这里特别需要注意，出现提示输入文件名的时候(`Enter file in which to save the key (~/.ssh/id_rsa): id_rsa_new`)要输入与默认配置不一样的文件名，比如：我这里填的是 `id_rsa`和`id_rsa_me`。

如果之前没配置过ssh key，这里用不同邮箱生成两遍即可，注意用不同的文件名

成功后会出现：

```shell
Your identification has been saved in xxx.
Your public key has been saved in xxx.
```

### 2.2 添加到ssh-agent中

使用`ssh-add`将 IdentityFile 添加到 ssh-agent中

```shell
ssh-add ~/.ssh/id_rsa
ssh-add ~/.ssh/id_rsa_me
```

### 2.3 配置 `~/.ssh/config` 文件

在`~/.ssh/`下新建`config`文件

```shell
# The git info for company
Host git.XXX.com				# git别名，写公司的git名字即可
HostName git.XXX.com				# git名字，同样写公司的git名字
User git					# 写 git 即可
IdentityFile ~/.ssh/id_rsa	        #私钥路径，若写错会连接失败

# The git info for github			
Host github.com					# git别名，写github的git名字即可
HostName github.com			        # git名字，同样写github的git名字
User git					# 写 git 即可
IdentityFile ~/.ssh/id_rsa_me	#私钥路径，若写错会连接失败
```



## 3 与GitHub链接

复制刚刚生成的两个ssh公钥到对应的账号中

文件`id_rsa.pub`中保存的就是 ssh 公钥

```shell
pbcopy < ~/.ssh/id_rsa.pub
pbcopy < ~/.ssh/id_rsa_me.pub
```

在 github 网站中添加该 ssh 公钥



验证是否配置成功，以 github 为例，输入 `ssh -T git@github.com`，若出现

```
Hi xxx! You've successfully authenticated, but GitHub does not provide shell access.
```

这样的字段，即说明配置成功。另一个同理。



> 参考链接：
>
> 配置多个git账号的ssh密钥 - 掘金
> https://juejin.im/post/5befe84d51882557795cc8f9
>
> 
>
> 同一台电脑配置多个git账号 · Issue #2 · jawil/notes
> https://github.com/jawil/notes/issues/2

