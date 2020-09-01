---
type: blog
title: 构建Hadoop的Docker编译环境
date: 2020-09-01
categories: 教程
tags: docker,教程
cover: false
meta:
  date: false
---



## 一、配置 docker 环境

> 参考链接：
> [Hadoop安装之一：使用Docker编译64位的Hadoop - 简书](https://www.jianshu.com/p/0d3c17b4dddb)

### 1. 制作 CentOS 7 基础镜像（可选）

Docker Hub上已经提供了[CentOS7的官方镜像](https://link.jianshu.com?t=https://hub.docker.com/_/centos/)，但并未激活 Systemd（用来启动守护进程），制作一个启动 Systemd 的镜像。（这里编译Hadoop其实用不到systemd）

- Dockerfile

```bash
# 镜像来源
FROM centos:7

# 镜像创建者
MAINTAINER "you" <your@email.here>

# 设置一个环境变量
ENV container docker

# 运行命令
# 设置systemd
RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == \
systemd-tmpfiles-setup.service ] || rm -f $i; done); \
rm -f /lib/systemd/system/multi-user.target.wants/*;\
rm -f /etc/systemd/system/*.wants/*;\
rm -f /lib/systemd/system/local-fs.target.wants/*; \
rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
rm -f /lib/systemd/system/basic.target.wants/*;\
rm -f /lib/systemd/system/anaconda.target.wants/*;

# 挂载一个本地文件夹
VOLUME [ "/sys/fs/cgroup" ]

# 设置容器启动时的执行命令
CMD ["/usr/sbin/init"]
```

- 生成镜像

```bash
docker build -t centos7-systemd .
```



### 2. 安装 Oracle Java

> 参考链接
> [使用yum卸载、安装jdk_不做小白的博客-CSDN博客](https://blog.csdn.net/weixin_40651304/article/details/78833642)

注意不要使用 openjdk，会导致编译 hive 时出现问题

- 启动刚刚生成的镜像
- 从官网下载 oracle java `jdk-8u202-linux-x64.tar.gz`
- 安装 Java，配置环境变量

```bash
mkdir /usr/local/java
cp jdk-8u202-linux-x64.tar.gz /usr/local/java
cd /usr/local/java
tar -xzvf jdk-8u202-linux-x64.tar.gz

# 配置环境变量
vim /etc/profile
~
~
export JAVA_HOME=/usr/local/java/jdk1.8.0_202
export JRE_HOME=/usr/local/java/jdk1.8.0_202/jre  
export PATH=$PATH:/usr/local/java/jdk1.8.0_202/bin  
export CLASSPATH=./:/usr/local/java/jdk1.8.0_202/lib:/usr/local/java/jdk1.8.0_202/jre/lib
~
~
source /etc/profile

# 检查 JAVA 是否安装成功
java -version
```

- 保存镜像

```bash
docker commit 容器id 镜像名
```



### 3. 制作编译镜像

- 编译脚本

```bash
$ vi compile.sh

#!/bin/bash

# 设置默认编译版本(支持传参)
version=${1:-2.7.3}

# 进入源代码目录
cd /hadoop-$version-src

# 开始编译
echo -e "\n\ncompile hadoop $version..."
mvn package -Pdist,native -DskipTests -Dtar

# 输出结果
if [[ $? -eq 0]]; then
 echo -e "\n\ncompile hadoop $version success!\n\n"
else
 echo -e "\n\ncompile hadoop $version fail!\n\n"
fi
```

- Dockerfile（其中有不少安装包不是必要的）

```bash
# 镜像来源(第二步生成的本地镜像)
FROM centos7-systemd-java

# 镜像创建者
MAINTAINER "you" <your@email.here>

# 运行命令安装环境依赖
# 使用 -y 同意全部询问
RUN yum update -y && \
    yum groupinstall -y "Development Tools" && \
    yum install -y wget \
               protobuf-devel \
               protobuf-compiler \
               maven \
               cmake \
               pkgconfig \
               openssl-devel \
               zlib-devel \
               gcc \
               automake \
               autoconf \
               make
               
# 复制编辑脚本文件到镜像中
COPY compile.sh /root/compile.sh
# 设置脚本文件的可运行权限
RUN chmod +x /root/compile.sh
```

- 生成镜像

```bash
sudo docker build -t centos7-hadoop-compiler .
```



## 二、编译源码

- hive（大概10分钟）

```bash
mvn clean package -Pdist -DskipTests
```

- hadoop（大概15分钟）

```bash
mvn clean package -Pdist,native -DskipTests -Dtar
```

也可以使用 docker image 中的脚本编译

```bash
$ export VERSION=2.7.3
$ sudo docker run -v $(pwd)/hadoop-$VERSION-src:/hadoop-$VERSION-src --privileged=true  centos7-hadoop-complier /root/compile.sh $VERSION
```

**要添加 privileged 参数！**

> [[docker]privileged参数_追寻神迹-CSDN博客](https://blog.csdn.net/halcyonbaby/article/details/43499409)

使用该参数，container内的root拥有真正的root权限。
否则，container内的root只是外部的一个普通用户权限。

hadoop-2.7.0-src.tar.gz  release-2.3.0.tar.gz



---

已包括各种库的 image，可以直接编译 hadoop

GitHub - kiwenlau/compile-hadoop: Compile Hadoop in Docker container
https://github.com/kiwenlau/compile-hadoop

