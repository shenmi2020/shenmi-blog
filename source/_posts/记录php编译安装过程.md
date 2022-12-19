---
title: 记录php编译安装过程
date: 2022-11-29 15:21:59
categories: [编程]
tags:
 - [php]
---


    1.在php官网下载页下载php的编译安装包，这里以7.4.28举例
    https://www.php.net/downloads.php
    找到php-7.4.28.tar.gz，右键复制下载链接，例如：
    https://www.php.net/distributions/php-7.4.28.tar.gz
    centos终端输入命令进行下载：
    wget https://www.php.net/distributions/php-7.4.28.tar.gz
    2.编译安装
        1.在下载目录进行解压
        tar -zxvf php-7.3.28.tar.gz
        2.进入解压包目录,这里完整目录名可以输入php后，点击tab按键快速自动补全
        cd php-7.3.28
        3.安装依赖
        yum install libxml2-devel sqlite-devel libcurl-devel oniguruma-devel libpng-devel libjpeg-devel freetype-devel libzip-devel openssl-devel -y
        设置编译参数。因为目录内含有configure文件，所以可进行编译操作，编译编译参考如下
        4.设置编译参数。因为目录内含有configure文件，所以可进行编译操作，编译编译参考如下
        ./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php/etc --with-mhash --with-openssl --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-iconv --with-zlib --enable-inline-optimization --disable-debug --disable-rpath --enable-shared --enable-bcmath --enable-shmop --enable-sysvsem --enable-gd --with-jpeg --with-freetype --enable-mbregex --enable-mbstring --enable-ftp --enable-pcntl --enable-sockets --enable-soap --without-pear --with-gettext --enable-session --with-curl  --enable-opcache --enable-fpm --with-fpm-user=php --with-fpm-group=php --without-gdbm --enable-fast-install --disable-fileinfo
        5.编译安装，这个过程比较久，需要耐心等待。
        make && make install