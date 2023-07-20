---
title: hyperf使用redis订阅
date: 2023-07-20 09:29:13
categories: [编程]
tags:
 - [hyperf]
 - [redis]
---
每60秒会抛出Redis::__call failed, because read error on connection异常
开始设置ini_set('default_socket_timeout', -1) 没有解决问题，default_socket_timeout是socket流的超时参数，即socket流从建立到传输再到关闭整个过程必须要在这个参数设置的时间以内完成，如果不能完成，那么PHP将自动结束这个socket并返回一个警告。这个参数也会影响file_get_content等方法。

最后设置OPT_READ_TIMEOUT为-1解决。
出现该问题的原因是poll设置接收超时所致，这个超时默认设置60s

``` php
<?php
declare(strict_types=1);

namespace App\Process;

use Hyperf\Process\AbstractProcess;
use Hyperf\Process\Annotation\Process;
use Psr\Container\ContainerInterface;
use Hyperf\Context\ApplicationContext;

#[Process(name: "foo_process")]
class FooProcess extends AbstractProcess
{

    public function handle(): void
    {   
        $container = ApplicationContext::getContainer();
        $redis = $container->get(\Hyperf\Redis\Redis::class);
        $redis->setOption(\Redis::OPT_READ_TIMEOUT, -1);
        var_dump('handle:' . $redis->getOption(\Redis::OPT_READ_TIMEOUT));
        $redis->subscribe(['r2'], function ($instance, $channelName, $message) {
            var_dump($instance);
            var_dump($channelName);
            var_dump($message);
            // TODO: 执行对应的消费操作
        });
    }
}
```