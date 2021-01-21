---
type: blog
title: CentOS6 yum 404
date: 2021-01-21
categories: 教程
tags: yum
cover: false
meta:
  date: false
---

## 一、问题出现原因

突然发现 yum不可用了，错误信息如下：

> Determining fastest mirrors
> YumRepo Error: All mirror URLs are not using ftp, http[s] or file.
> Eg. Invalid release/repo/arch combination/
> removing mirrorlist with no valid mirrors: /var/cache/yum/x86_64/6/base/mirrorlist.txt
> 错误：Cannot find a valid baseurl for repo: base

前面怀疑是服务器的网络问题，经排查网络无异常。拿yum源中的地址确认问题，发现404了，地址已经发发生了改变，原因是CentOS 6已经随着2020年11月的结束进入了EOL（Reaches End of Life），官方便在12月2日正式将CentOS 6相关的软件源移出了官方源，随之而来逐级镜像也会陆续将其删除。

## 二、解决问题

**备份文件**

```bash
# by root
cp /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
```

**替换CentOS-Base.repo文件内容**

下面提供了官方Vault源和阿里云Vault镜像，选择其一即可，国内建议使用阿里云Vault镜像，速度会更快。
`vi /etc/yum.repos.d/CentOS-Base.repo`

```
#阿里云Vault镜像，本例使用的6.10版本，注意修改为你当前的操作系统版本号
[base]
name=CentOS-6.10 - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos-vault/6.10/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos-vault/RPM-GPG-KEY-CentOS-6
 
#released updates 
[updates]
name=CentOS-6.10 - Updates - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos-vault/6.10/updates/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos-vault/RPM-GPG-KEY-CentOS-6
 
#additional packages that may be useful
[extras]
name=CentOS-6.10 - Extras - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos-vault/6.10/extras/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos-vault/RPM-GPG-KEY-CentOS-6
 
#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-6.10 - Plus - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos-vault/6.10/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos-vault/RPM-GPG-KEY-CentOS-6
 
#contrib - packages by Centos Users
[contrib]
name=CentOS-6.10 - Contrib - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos-vault/6.10/contrib/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos-vault/RPM-GPG-KEY-CentOS-6



--官方Vault源，本例使用的6.10版本，注意修改为你当前的操作系统版本号
[base]
name=CentOS-6.10 - Base - vault.centos.org
failovermethod=priority
baseurl=http://vault.centos.org/6.10/os/$basearch/
gpgcheck=1
gpgkey=http://vault.centos.org/RPM-GPG-KEY-CentOS-6
 
#released updates 
[updates]
name=CentOS-6.10 - Updates - vault.centos.org
failovermethod=priority
baseurl=http://vault.centos.org/6.10/updates/$basearch/
gpgcheck=1
gpgkey=http://vault.centos.org/RPM-GPG-KEY-CentOS-6
 
#additional packages that may be useful
[extras]
name=CentOS-6.10 - Extras - vault.centos.org
failovermethod=priority
baseurl=http://vault.centos.org/6.10/extras/$basearch/
gpgcheck=1
gpgkey=http://vault.centos.org/RPM-GPG-KEY-CentOS-6
 
#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-6.10 - Plus - vault.centos.org
failovermethod=priority
baseurl=http://vault.centos.org/6.10/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://vault.centos.org/RPM-GPG-KEY-CentOS-6
 
#contrib - packages by Centos Users
[contrib]
name=CentOS-6.10 - Contrib - vault.centos.org
failovermethod=priority
baseurl=http://vault.centos.org/6.10/contrib/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://vault.centos.org/RPM-GPG-KEY-CentOS-6
```

**清除YUM缓存**

```bash
yum clean all
```

**重新构建缓存**

```bash
yum makecache
```

