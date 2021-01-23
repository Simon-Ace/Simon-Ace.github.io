---
type: blog
title: raw.githubusercontent.com无法连接
date: 2021-01-24
categories: 教程
tags: 教程
cover: false
meta:
  date: false
---



由于 DNS 污染导致 raw.githubusercontent.com 无法正常访问，可通过在 hosts 中添加下面一行解决：

```
199.232.96.133 raw.githubusercontent.com
```

<!-- more -->



## 一、解决方法

**查询真实IP**

通过[`IPAddress.com`](https://www.ipaddress.com/)首页，输入`raw.githubusercontent.com`查询到真实IP地址

如查询到的ip为：`199.232.96.133`

**修改hosts**

```
sudo vi /etc/hosts
```

添加以下内容保存

```
199.232.96.133 raw.githubusercontent.com
```



## 二、其他代理

可参考 [`https://ghproxy.com`](https://ghproxy.com/) [更详细使用方法](https://www.ioiox.com/archives/102.html)

主要用于 clone github 的项目