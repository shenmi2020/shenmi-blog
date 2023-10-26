---
title: nginx-failed错误
date: 2023-10-26 10:39:45
tags:
 - [nginx]
---
发现nginx错误日志里写满了connect() failed (111: Connection refused) while connecting to upstream，但是请求正常，php-fpm也正常，查询后找到原因：
::1指的是本地ipv6地址
proxy_pass中如果是localhost则会转发给ipv4和ipv6，所以会出现上面的错误日志；
改成127.0.0.1问题解决。
./nginx -s reload