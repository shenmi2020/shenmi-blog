---
title: docker常用命令记录
date: 2024-01-02 10:22:46
categories: [编程]
tags:
 - [docker]
---

<!-- more -->

``` shell
# 导出容器 (发现挂载的目录内文件导出时没有保存在容器中
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
docker run -it -v /var/www/workspace:/var/www/workspace -p 8989:8989 --name new_name images_name /bin/bash
--restart=always 自启动

#生成镜像
docker commit -a shenmi -m php7.4-fpm php7.4-fpm wx/php:7.4.33-fpm


# 经测试上面这行命令在 CentOS 7 下目录挂载失败。
# 在上面这行命令的基础上增加了--privileged=true参数，让容器拥有真正的root权限
docker run --privileged=true --name mysql5.7 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d  -v /usr/local/docker_data/mysql/data:/var/lib/mysql -v /usr/local/docker_data/mysql/conf:/etc/mysql/ -v /usr/local/docker_data/mysql/logs:/var/log/mysql mysql:5.7