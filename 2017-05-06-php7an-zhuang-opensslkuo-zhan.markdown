---
layout: post
title: "PHP7安装OpenSSL扩展"
date: 2015-11-21 23:05:31 +0800
comments: true
categories: Dev
tags: [PHP]
---


<!--more-->
因为项目中需要发邮件，使用了`ssl`加密，所以需要`PHP`对`OpenSSL`支持。没有这个扩展的话，会报这个错误。
```
Unable to find the socket transport “ssl” - did you forget to enable it when you configured PHP
```
首先查找PHP中是否已经有这个扩展，是否未加载成功。
```
$ php-config70 --extension-dir  
/opt/local/lib/php70/extensions/no-debug-non-zts-20141001
$ cd /opt/local/lib/php70/extensions/no-debug-non-zts-20141001
```

如果没有安装，目录下一版没有`openssl.so`这个文件。一种简单的安装方法，是从`PHP`的源码重新编译这个扩展。 
到官网[https://downloads.php.net/~ab/](https://downloads.php.net/~ab/)下载对应版本的`PHP7`源码.
```
$ wget https://downloads.php.net/~ab/php-7.0.0RC6.tar.gz
$ tar zxvf php-7.0.0RC6.tar.gz
$ cd php-7.0.0RC6/ext/openssl
```
在目录下下执行`phpize70`,会报错。提示使用绝对路径
```
$ /opt/local/bin/phpize70
Cannot find config.m4  # 报错找不到文件

# 然后执行
$ mv config0.m4 config.m4
# 再次执行 /opt/local/bin/phpize70
Configuring for:
PHP Api Version:         20131218
Zend Module Api No:      20141001
Zend Extension Api No:   320140815

## 编译安装,首先找到php-config70文件路径
$ where php-config70
/opt/local/bin/php-config70
$ ./configure --with-php-config=/opt/local/bin/php-config70
$ sudo make & make install
```
安装成功之后，出现
```
/bin/sh /Users/ilovey/Downloads/php-7.0.0RC6/ext/openssl/libtool --mode=install cp ./openssl.la /Users/ilovey/Downloads/php-7.0.0RC6/ext/openssl/modules
cp ./.libs/openssl.so /Users/ilovey/Downloads/php-7.0.0RC6/ext/openssl/modules/openssl.so
cp ./.libs/openssl.lai /Users/ilovey/Downloads/php-7.0.0RC6/ext/openssl/modules/openssl.la
----------------------------------------------------------------------
Libraries have been installed in:
   /Users/ilovey/Downloads/php-7.0.0RC6/ext/openssl/modules

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the `-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the `DYLD_LIBRARY_PATH' environment variable
     during execution

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------
Installing shared extensions:     /opt/local/lib/php70/extensions/no-debug-non-zts-20141001/
```
此时，会发现`/opt/local/lib/php70/extensions/no-debug-non-zts-20141001/`目录下多了个`openssl.so`的文件，我们让`PHP`将其加载就行.
加载的方式有两种，一种直接在`php.ini`文件中添加`extension=openssl.so`

另一种方式，查找到PHP配置文件目录
```
$ php-config70 --configure-options
可以找到如下
--with-config-file-scan-dir=/opt/local/var/db/php70
```
进入可以看到有数个`ini`文件。添加一个`openssl.ini`文件，并在其中添加`extension=openssl.so`.

重启`php-fpm`,就可以看到扩展加载成功了。

转载自[php开启openssl的方法](http://www.52jscn.com/web/2013/05/4592.shtml)

