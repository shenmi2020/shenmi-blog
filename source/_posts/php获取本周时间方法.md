---
title: php获取本周时间方法
date: 2021-01-06 15:22:54
categories: [编程]
tags:
 - [php]
---

``` php
// 获取本周时间
$nowDate = date('Y-m-d');
$week = date('w', strtotime($nowDate));
$start_time = strtotime("$nowDate - " . ($week ? $week - 1 : 6).' days');
// 本月时间
$start_time = date('Y-m-01');
```