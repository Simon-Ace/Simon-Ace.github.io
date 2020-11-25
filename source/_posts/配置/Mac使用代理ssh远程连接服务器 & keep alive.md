---
type: blog
title: Mac使用代理ssh远程连接服务器 & keep alive
date: 2020-07-10
categories: 教程
tags: 教程
cover: false
meta:
  date: false
---



```bash
# 直接连接
ssh -p 端口号 服务器用户名@ip地址
# eg: ssh -p 22 userkunyu@119.29.37.63

# 通过代理连接
ssh -o ProxyCommand="nc -X 5 -x 代理服务器ip:代理服务器端口 %h %p" 需要访问的服务器的用户名@需要访问的服务器ip
```



<!-- more -->



## 1 直接连接

```bash
# ssh -p 端口号 服务器用户名@ip地址
ssh -p 22 userkunyu@119.29.37.63
```



## 2 通过代理连接

1. 直接连接

```bash
# ssh -o ProxyCommand="nc -X 5 -x 代理服务器ip:代理服务器端口 %h %p" 需要访问的服务器的用户名@需要访问的服务器ip
ssh -o ProxyCommand="nc -X 5 -x 192.168.0.255:9999 %h %p" user_name@192.168.77.200
```

2. 使用SSH配置文件

```bash
sudo vi ~/.ssh/config
```

```
Host *
    ProxyCommand nc -X 5 -x 192.168.0.255:9999 %h %p
```

配置好了之后就可以和直接连接一样使用

```bash
ssh uesr@ip
```



> Mac下SSH跳点连接及代理连接_Dawnworld-CSDN博客_mac ssh 代理
> https://blog.csdn.net/thundon/article/details/46858957



## 3 Keep alive

> http://bluebiu.com/blog/iterm2-ssh-session-idle.html

**方案一：**

在本机 `vim ~/.ssh/config`

```
# 在开头添加
Host *
    ServerAliveInterval 60
```

我觉得60秒就好了，而且基本去连的机器都保持，所以配置了`*`，如果有需要针对某个机器，可以自行配置为需要的`serverHostName`。



**方案二：**

单次连接

添加下面的参数

```bash
ssh -o ServerAliveInterval=30 user@host
```

