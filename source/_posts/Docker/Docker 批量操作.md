---
type: blog
title: Docker 批量操作
date: 2020-08-28
categories: 技术
tags: docker
cover: false
meta:
  date: false
---



```bash
docker rmi $(docker images | grep "none" | awk '{print $3}') 
```

<!-- more -->

- 列出所有的容器 ID

```bash
docker ps -aq
```

- 停止所有的容器

```bash
docker stop $(docker ps -aq)
```

- 删除所有的容器

```bash
docker rm $(docker ps -aq)
```

- 删除所有的镜像

```bash
docker rmi $(docker images -q)
```

- 删除指定名称镜像

```bash
docker rmi $(docker images | grep "none" | awk '{print $3}') 
```

- 复制文件

```
docker cp mycontainer:/opt/file.txt /opt/local/
docker cp /opt/local/file.txt mycontainer:/opt/
```



现在的docker有了专门清理资源(container、image、网络)的命令。 

docker 1.13 中增加了 `docker system prune`的命令，针对container、image可以使用`docker container prune`、`docker image prune`命令。

- 删除所有不使用的镜像

```bash
docker image prune --force --all
# or
docker image prune -f -a
```

- 删除所有停止的容器

```bash
docker container prune -f
```

