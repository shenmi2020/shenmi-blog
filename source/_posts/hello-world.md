---
title: Hello World
categories: [编程]
---
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## Quick Start

### Create a new post

``` php
public function test()
{
    // 初始数据
    $items = array(
        array('id' => 1, 'pid' => 0, 'name' => '福建省'),
        array('id' => 2, 'pid' => 0, 'name' => '四川省'),
        array('id' => 3, 'pid' => 1, 'name' => '福州市'),
        array('id' => 4, 'pid' => 2, 'name' => '成都市'),
        array('id' => 5, 'pid' => 2, 'name' => '乐山市'),
        array('id' => 6, 'pid' => 4, 'name' => '成华区'),
        array('id' => 7, 'pid' => 4, 'name' => '龙泉驿区'),
        array('id' => 8, 'pid' => 6, 'name' => '崔家店路'),
        array('id' => 9, 'pid' => 7, 'name' => '龙都南路'),
        array('id' => 10, 'pid' => 8, 'name' => 'A店铺'),
        array('id' => 11, 'pid' => 9, 'name' => 'B店铺'),
        array('id' => 12, 'pid' => 8, 'name' => 'C店铺'),
        array('id' => 13, 'pid' => 1, 'name' => '泉州市'),
        array('id' => 14, 'pid' => 13, 'name' => '南安县'),
        array('id' => 15, 'pid' => 13, 'name' => '惠安县'),
        array('id' => 16, 'pid' => 14, 'name' => 'A镇'),
        array('id' => 17, 'pid' => 14, 'name' => 'B镇'),
        array('id' => 18, 'pid' => 16, 'name' => 'A村'),
        array('id' => 19, 'pid' => 16, 'name' => 'B村'),
    );
 
    // 根据初始数据，生成一个以 id 为 key/下标 的数组，方便根据 pid 判断是否存在父级元素。
    $items = array_column($items,null,'id');
 
    //使用 php 的 & 引用特性，遍历一次循环即可生成无限级分类树。（其他高级语言中也有类似的特性，诸如 C++ 的指针和 JAVA 的引用）

    $tree = [];
    foreach ($items as $item) {
        $id = $item['id'];
        $pid = $item['pid'];
 
        if (isset($items[$pid]))
            $items[$pid]['children'][] = &$items[$id];
        else
            $tree[] = &$items[$id];
    }

}
```

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)
