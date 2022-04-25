---
title: php使用exec函数编译c++报错collect2
date: 2019-06-17 15:28
categories: [编程]
tags:
 - [php]
---

使用exec函数编译c++报错collect2: fatal error: cannot find 'ld'

<!-- more -->

解决方案
配置环境变量后解决

/etc/php/7.0/fpm/pool.d/www.conf

![](https://i.imgtg.com/2022/04/25/xdcOb.png)

重启fpm后编译命令生效

命令g++ spj.cc -o spj 2>&1成功执行0
