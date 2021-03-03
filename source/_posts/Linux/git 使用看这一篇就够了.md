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

