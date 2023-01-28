---
title: 解决Linux下杀死进程又自启动问题
date: 2023-01-28 17:15:22
tags:
 - [ubuntu]
---
1. ps -aux 查找到对应进程pid号

2. cd /proc/PID号  

3. cat status, 找到父进程的PPID号

4. 先杀掉父进程
kill -9 PPID号

5. 再杀掉子进程
kill -9 PID号