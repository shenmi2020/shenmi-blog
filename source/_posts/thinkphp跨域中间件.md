---
title: thinkphp跨域中间件
date: 2022-10-20 10:18:49
categories: [编程]
tags:
 - [thinkphp]
 - [php]
---

thinkphp框架跨域中间件代码记录
也可以使用tp官方中间件    \think\middleware\AllowCrossDomain::class

<!-- more -->

``` php
<?php
declare (strict_types = 1);
 
namespace app\wechat\middleware;
use think\Response;
 
/**
 * 全局跨域请求处理
 * Class CrossDomain
 * @package app\middleware
 */
 
class CrossDomain
{
    public function handle($request, \Closure $next)
    {
        header('Access-Control-Allow-Origin: *');
        header('Access-Control-Max-Age: 1800');
        header('Access-Control-Allow-Methods: GET, POST, PATCH, PUT, DELETE');
        // header('Access-Control-Allow-Headers: Authorization, Content-Type, If-Match, If-Modified-Since, If-None-Match, If-Unmodified-Since, X-CSRF-TOKEN, X-Requested-With, Token, Cid, Clent-info');
        header('Access-Control-Allow-Headers: *');
        if (strtoupper($request->method()) == "OPTIONS") {
            return Response::create()->send();
        }
 
        return $next($request);
    }
}
```