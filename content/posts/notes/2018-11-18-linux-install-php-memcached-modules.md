---
id: 1787
title: 'Linux 系统安装 PHP memcached 扩展模块'
date: '2018-11-18T06:25:02+08:00'
author: hacper
layout: post
guid: 'https://hacperme.com/?p=1787'
permalink: /2018/11/18/linux-install-php-memcached-modules/
views:
    - '1221'
categories:
    - 笔记
tags:
    - linux
    - memcached
    - PHP
---

下载源码：

`wget http://pecl.php.net/get/memcached-3.0.4.tgz`

解压：

`tar -zxf memcached-3.0.4.tgz`

切换到解压目录：

`cd memcached-3.0.4`

创建 configure 文件：

`phpize`

编译配置：

`./configure --with-php-config=/www/server/php/72/bin/php-config  --with-libmemcached-dir=/usr/local/libmemcached`

配置参数中 php-config 和 libmemcached 的路径可以用find命令查找：

`find / -name php-config`  
`find / -name libmemcached`

编译：

`make`

安装：

`make install`

修改配置文件 php.ini， 添加模块 extension = memcache.so：

`find / -name php.ini`

`vim /www/server/php/72/etc/php.ini`

`extension = memcache.so`

重启 PHP 后，查看PHP模块是否启用：

`php -m`

```shell
[PHP Modules]
bcmath
mbstring
memcached
mysqli
pcre
PDO

...
...

xml
xmlreader
xmlrpc
xmlwriter
zip
zlib

[Zend Modules]

```