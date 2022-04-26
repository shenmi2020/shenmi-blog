---
title: hyperf框架cache类封装
date: 2022-01-16 11:12
categories: [编程]
tags:
 - [php]
 - [hyperf]
---

缓存调用封装
<!-- more -->
``` php
<?php

namespace App\Utils;

use Closure;
use Hyperf\Utils\ApplicationContext;
use Psr\SimpleCache\CacheInterface;

class Cache
{
    public static function getInstance()
    {
        return ApplicationContext::getContainer()->get(CacheInterface::class);
    }

    public static function remember($key, $ttl, Closure $callback)
    {
        $instance = self::getInstance();

        $value = $instance->get($key);

        if (!is_null($value)) {
            return $value;
        }

        $instance->set($key, $value = $callback(), $ttl);

        return $value;
    }

    public static function __callStatic($name, $arguments)
    {
        return self::getInstance()->$name(...$arguments);
    }
}
```

# DEMO

``` php
$user_city = Cache::remember("user_city|uid:{$user['id']}", 3600, function () use ($user) {
    return UserCityModel::where('uid', $user['id'])->orderBy('type')->get();
});
```
