---
title: docker常用命令记录
date: 2024-01-02 10:22:46
categories: [编程]
tags:
 - [docker]
---

<!-- more -->

``` shell
# 导出容器 (发现挂载的目录导出时没有保存在容器中
docker export container_name -o container_name.tar

# 导入容器 变成镜像
docker import ./container_name.tar test/ubuntu:v1.0

# 查看容器
docker container inspect 

# 导出镜像
docker save -o fedora-all.tar fedora

# 导入镜像 不能对载入的镜像重命名
docker load

# 查看镜像
docker images

# 查看容器
docker ps -a

# 运行容器
docker run -v /var/www/workspace:/var/www/workspace -p 8989:8989 --name new_name images_name
```
