---
title: docker安装php镜像和扩展
date: 2022-08-10 11:48:01
categories: [编程]
tags:
 - [php]
 - [docker]
---

<!-- more -->
因工作需要使用两种php版本，考虑使用docker来部署环境，几年前简单学习过docker现在已经忘干净了，再来学习下

先安装好docker，这里不做记录
1.去https://hub.docker.com/_/php/tags查询自己需要的版本，复制命令
docker pull php:8.2-rc-cli-buster
ps: 下载慢可以配置docker镜像，如阿里云和163等

2.docker run -t -i php:8.2 /bin/bash
这里最好配置好端口映射-p: 指定端口映射，格式为：主机(宿主)端口:容器端口
目录映射-v /data:/data
也可以后期修改docker配置文件，不过有点麻烦。。。

3.docker exec -it php:8.2 /bin/bash
进入运行的容器中，可以执行命令

到这里就可以使用了
如果需要连接宿主机的软件如mysql redis等，一般宿主机默认ip 172.17.0.1
记录下一些常用命令
 
```

docker start *  // 启动一个容器
docker images   // 查看镜像列表
docker ps -a    // 查看所有容器
```


vscode可以搜索安装docker插件，可以图形化启动容器，也可以编辑器终端打开容器shell

docker容器添加新的端口映射方法
/var/lib/docker/containers/<容器Id>/hostconfig.json
```
{
    "PortBindings":{
			"22/tcp":[{"HostIp":"","HostPort":"10112"}],
			"5901/tcp":[{"HostIp":"","HostPort":"10113"}],
	},
}

/var/lib/docker/containers/<容器Id>/config.v2.json
```
{
    "ExposedPorts":{
        "22/tcp":{},
        "5901/tcp":{}
    }
}
```

docker安装php扩展

在php容器中创建/usr/src/php目录
docker-php-source extract

docker-php-ext-configure
docker-php-ext-enable  启用扩展
docker-php-ext-install 安装扩展

