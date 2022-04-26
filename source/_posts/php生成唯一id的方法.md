---
title: php生成唯一id的方法
date: 2021-06-06 11:34
categories: [编程]
tags:
 - [php]
---

记录php生成唯一值的几种方法
<!-- more -->
``` php
/**
* 获取全球唯一标识
* @return string
*/
public static function uuid()
{
    return sprintf(
        '%04x%04x-%04x-%04x-%04x-%04x%04x%04x',
        mt_rand(0, 0xffff),
        mt_rand(0, 0xffff),
        mt_rand(0, 0xffff),
        mt_rand(0, 0x0fff) | 0x4000,
        mt_rand(0, 0x3fff) | 0x8000,
        mt_rand(0, 0xffff),
        mt_rand(0, 0xffff),
        mt_rand(0, 0xffff)
    );
}

// 第二种方法

$id = md5(uniqid(microtime(true),true));

```