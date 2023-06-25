---
title: thinkphp队列超时配置
date: 2023-06-25 16:09:21
tags:
 - [thinkphp]
---

使用tp6框架的queue队列时，如果执行时间超过60s，进程会自动kill掉。
设置参数timeout=0 解决
php think queue:work --queue queuename --sleep 0 --timeout 2048

部分参数记录
--memory 128  //该进程允许使用的内存上限，以 M 为单位
--sleep 3  //如果队列中无任务，则sleep多少秒后重新检查(work+daemon模式)或者退出(listen或非daemon模式)
--timeout 60 //创建的work子进程的允许执行的最长时间，以秒为单位
--delay 0 //如果本次任务执行抛出异常且任务未被删除时，设置其下次执行前延迟多少秒,默认为0