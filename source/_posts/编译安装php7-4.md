---
title: 编译安装php7.4
date: 2024-03-07 14:35:36
tags:
---
``` shell
./configure \
--prefix=/usr/local/php \
--with-config-file-path=/usr/local/php/etc \
--with-fpm-user=www \
--with-fpm-group=www \
--with-curl \
--with-freetype-dir \
--enable-gd \
--with-gettext \
--with-iconv-dir \
--with-kerberos \
--with-libdir=lib64 \
--with-libxml-dir \
--with-mysqli \
--with-openssl \
--with-pcre-regex \
--with-pdo-mysql \
--with-pdo-sqlite \
--with-pear \
--with-png-dir \
--with-jpeg-dir \
--with-xmlrpc \
--with-xsl \
--with-zlib \
--with-bz2 \
--with-mhash \
--enable-fpm \
--enable-bcmath \
--enable-libxml \
--enable-inline-optimization \
--enable-mbregex \
--enable-mbstring \
--enable-opcache \
--enable-pcntl \
--enable-shmop \
--enable-soap \
--enable-sockets \
--enable-sysvsem \
--enable-sysvshm \
--enable-xml \
--enable-zip \
--enable-fpm \
--enable-fileinfo
```