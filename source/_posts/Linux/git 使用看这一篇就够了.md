---
type: blog
title: git 使用看这一篇就够了
date: 2021-02-08
categories: 教程
tags: git
cover: false
meta:
  date: false
---
## 一、基础概念

区域

- 工作区
- 暂存区
- 版本库

git 对象



## 二、基本操作

### 2.1 初始化仓库

```bash
git init
```



## 三、进阶操作

- 同步另一个分支的修改

在 A、B 分支之前处于一个位置，之后A修改了，变成A'，想把B同步到A'的位置

```bash
# 先切换到 B 分支
git checkout B
# 合并 A1 的修改
git merge A
```

- 删除某次 commit

### 3.1 git reset

- `git reset` ：回滚到某次提交。
- `git reset --soft`：此次提交之后的修改会被退回到暂存区。
- `git reset --hard`：此次提交之后的修改不做任何保留，`git status` 查看工作区是没有记录的。

```cpp
git log // 查询要回滚的 commit_id
git reset --hard commit_id // HEAD 就会指向此次的提交记录
git push origin HEAD --force // 强制推送到远端
```

### 3.2 rebase

- 修改之前的commit信息（只允许在推到远程之前做）

```bash
git rebase -i <要改的commit 之前的一次commit id>
```

【原理】

在要做变基的节点上，创建出了新的一条支，然后在新的这个支上作修改。