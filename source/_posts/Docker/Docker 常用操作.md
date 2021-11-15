---
type: blog
title: Docker 常用操作
date: 2021-07-16
categories: 技术
tags: docker
cover: false
meta:
  date: false
---



<!-- more -->

Docker从容器内拷贝文件到主机上

```bash
docker cp <containerId>:/file/path/within/container /host/path/target
```



#### 进入容器

`exec` 命令

`-i` `-t` 参数

`docker exec` 后边可以跟多个参数，这里主要说明 `-i` `-t` 参数。

只用 `-i` 参数时，由于没有分配伪终端，界面没有我们熟悉的 Linux 命令提示符，但命令执行结果仍然可以返回。

当 `-i` `-t` 参数一起使用时，则可以看到我们熟悉的 Linux 命令提示符。

```
$ docker run -dit ubuntu
69d137adef7a8a689cbcb059e94da5489d3cddd240ff675c640c8d96e84fe1f6


$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
69d137adef7a        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                           zealous_swirles


$ docker exec -i 69d1 bash
ls
bin
boot
dev
...


$ docker exec -it 69d1 bash
root@69d137adef7a:/#
```

如果从这个 stdin 中 exit，不会导致容器的停止。这就是为什么推荐大家使用 `docker exec` 的原因。

更多参数说明请使用 `docker exec --help` 查看。



`docker attach`

理解上进入的原来的终端，没有生成新的终端（所以这种情况下，退出容器会导致容器退出）



#### volume 映射

```bash
# 前面是宿主机的目录，后面是容器内的目录
-v /host-dir:/docker-dir
```



#### 使用宿主机网络

```bash
--network host
```



#### 容器自动重启

```bash
--restart=always
```



#### 退出但不停止容器

```bash
Ctrl+P+Q
```



#### 替换 `entrypoint.sh`

```bash
# 替换entrypoint 参考
docker run -d centos /bin/bash -c "nohup ping -i 1000 www.baidu.com"
```





> [docker从入门到实践](https://yeasy.gitbook.io/docker_practice/container/attach_exec)
