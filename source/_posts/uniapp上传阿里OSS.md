---
title: uniapp上传阿里OSS
date: 2022-12-19 11:59:12
categories: [编程]
tags:
 - [uniapp]
---

开发过程中需要uniapp上传图片到阿里云oss，uniapp官方文档文件上传不支持PUT方式
搜索解决方案并记录
Uniapp 通过 get 请求获取文件字节流后，再使用PUT请求上传文件
``` js
uni.request({
    url: path, //仅为示例，并非真实接口地址。
    responseType: 'arraybuffer',
    success: (res) => {
        console.log(res);
    }
});
```