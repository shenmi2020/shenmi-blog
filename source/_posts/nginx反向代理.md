---
title: nginx反向代理
categories: [编程]
date: 2021-04-22
tags:
 - nginx
---

记录nginx在二级目录下设置反向代理

<!-- more -->

``` nginx
upstream wxrecord {
    server 127.0.0.1:9001;
}

location /api/ {
    # First attempt to serve request as file, then
    # as directory, then fall back to displaying a 404.
    #try_files $uri $uri/ =404;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    # 转发Cookie，设置 SameSite
    proxy_cookie_path / "/; secure; HttpOnly; SameSite=strict";

    # 执行代理访问真实服务器
    proxy_pass http://wxrecord/;
}
```
注意：反向代理设置在二级目录时，如果proxy_pass后有/表示绝对根路径。如果没有/，表示相对路径，把匹配的路径部分也给代理走。
