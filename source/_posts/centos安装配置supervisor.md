---
title: centos安装配置supervisor
date: 2023-04-21 16:02:45
tags:
 - [centos]
---

经常使用ubuntu,对centos的命令不熟悉，记录下在服务器安装supervisor的过程

<!-- more -->

## 名词解释

```javascript
supervisor：要安装的软件的名称。
supervisord：装好supervisor软件后，supervisord用于启动supervisor服务。
supervisorctl：用于管理supervisor配置文件中program

```

## 安装依赖

```
yum install epel-release

```

## 安装supervisor

```
yum install -y supervisor

```

## 开机自启

```
systemctl enable supervisord

```

## 启动supervisord

```
systemctl start supervisord

```

## 查看状态

    systemctl status supervisord

## 修改配置

```
vim /etc/supervisord.conf

```

```
# 调整增加引入配置文件路径，这个路径放置项目对应的 supervisor 配置文件
# include表示/etc/supervisord.d/文件夹下的所有的.ini文件，都作为启动的应用程序（下文简称“进程”）
# 每一个.ini对应一个应用程序的进程，包括但不限于dotnet应用进程
[include]
files = /etc/supervisord.d/*.ini


```

## supervisorctl常用命令

    supervisorctl status           #查看程序状态
    supervisorctl stop name        #关闭name程序
    supervisorctl start name       #启动name程序
    supervisorctl restart name     # 重启name程序
    supervisorctl reread           #读取有更新的配置文件，不会启动新添加的程序
    supervisorctl update           #重启配置文件修改过的程序

```
# 新建一个应用并设置一个名称，这里设置为 hyperf
[program:hyperf]
# 设置命令在指定的目录内执行
directory=/home/king/workspace/hyperf-skeleton/
# 这里为您要管理的项目的启动命令
command=php ./bin/hyperf.php start
# 以哪个用户来运行该进程
user=root
# supervisor 启动时自动该应用
autostart=true
# 进程退出后自动重启进程
autorestart=true
# 进程持续运行多久才认为是启动成功
startsecs=1
# 重试次数
startretries=3
# stderr 日志输出位置
stderr_logfile=/var/www/log/hyperf/stderr.log
# stdout 日志输出位置
stdout_logfile=/var/www/log/hyperf/stdout.log
```


