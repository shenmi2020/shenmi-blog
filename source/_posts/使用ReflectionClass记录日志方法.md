---
title: 使用ReflectionClass记录日志方法
date: 2024-01-17 11:12:50
categories: [编程]
tags:
 - [php]
---
<!-- more -->
记录一下啊使用php反射特性来记录操作日志

``` php

$re = new ReflectionClass(CLASS);
$doc = $re->getMethod($request->action())->getDocComment();
if (empty($doc)) {
    throw new Exception('请给控制器方法注释');
}
preg_match('/\s(\w+)/u', $re->getMethod($request->action())->getDocComment(), $values);
$notes = $values[0];


/**
 * 测试操作
 */
public function test() {
    echo "test";
}

```