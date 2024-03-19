---
title: docker安装php镜像和扩展
date: 2022-08-10 11:48:01
categories: [编程]
tags:
 - [php]
 - [docker]
---


先安装好docker，这里不做记录
1.去https://hub.docker.com/_/php/tags查询自己需要的版本，复制命令
docker pull php:8.2-rc-cli-buster
ps: 下载慢可以配置docker镜像，如阿里云和163等

<!-- more -->

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

pecl install redis
docker-php-ext-enable redis
------
checking for libpng... no
configure: error: Package requirements (libpng) were not met:
 
No package 'libpng’ found

apt-get install libpng-dev

configure: error: Package requirements (zlib >= 1.2.0.4) were not met:
 
No package 'zlib’ found 

apt-get install zlib1g-dev

----
kill -USR2 1  容器内重启php-fpm

fastcgi_param SCRIPT_FILENAME /var/www/html$fastcgi_script_name;  // nginx修改配置

// 安装composer
curl -sS https://getcomposer.org/installer | php
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/


docker run -itd --name php7.4-fpm-c2  -v /var/www/wwwdata:/var/www/html -v /var/docker/php/etc:/usr/local/etc -v /var/docker/php/supervisor/conf.d:/etc/supervisor/conf.d -v /var/docker/php/supervisor/log:/var/www/log -p 19001:9000 --privileged=true --restart=always weixiao/php:7.4.33-fpm /bin/bash


server {
        listen 8081;
        listen [::]:8081;

        server_name _;

        root /var/www/wwwdata/cloud_school_php/public;
        index index.html index.php;

        location / {
                #if (!-e $request_filename) {
                #       rewrite  ^(.*)$  /index.php?s=/$1  last;
                #}
            if (!-e $request_filename) {
                rewrite  ^(.*)$  /index.php?s=$1  last;
                break;
            }
        }

        location ~ \.php$ {
                fastcgi_index index.php;
                root /var/www/html/cloud_school_php/public;
                #include snippets/fastcgi-php.conf;
                fastcgi_split_path_info ^(.+\.php)(.*)$;
                fastcgi_param PATH_INFO $fastcgi_path_info;   #使nginx支持pathinfo
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
               
                include  fastcgi_params;
               
                fastcgi_pass 127.0.0.1:19000;
        }
}