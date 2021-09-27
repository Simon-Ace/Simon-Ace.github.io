---
type: blog
title: logrotate 日志管理
date: 2021-09-23
categories: 技术
tags: log
cover: false
meta:
  date: false
---



<!-- more -->

```bash
vi /etc/logrotate.d/xxx  #根据对应需求写文件名
```

```bash
/xx/xx/xxx.log {
    missingok
    notifempty
    maxsize 10G
    daily
    create 0600 root root
    rotate 10
    compress
    copytruncate
    dateext
}
```

系统会自动执行，对相应的log文件进行 rotate 处理



> https://www.cnblogs.com/wushuaishuai/p/9330952.html