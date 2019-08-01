---
title: 在PHPStorm中使用XDebug调试PHP
date: 2018-02-25 00:00:00
tags: ["php"]
abbrlink: phpstorm-debug-with-xdebug-extension
img: ""
comments: false
---

之前一直通过echo，var_dump，print_r等将变量输出来调试PHP，效率奇低。而使用xdebug，就可以直接在IDE中调试PHP了。
## 安装XDEBUG
### 下载xdebug

请下载对应PHP版本的xdebug
```php
wget wget http://xdebug.org/files/xdebug-2.2.1.tgz
tar xzvf xdebug-2.2.1.tgz
cd xdebug-2.2.1
```



### 安装xdeubg
```php
/usr/local/webserver/php/bin/phpize
./configure --enable-xdebug --with-php-config=/usr/local/webserver/php/bin/php-config
make
cp modules/xdebug.so /usr/local/webserver/php/lib/php/extensions/
//修改 php.ini
vi /usr/local/webserver/php/etc/php.ini
```

在最底下加入以下内容：
```ini
[XDEBUG]
zend_extension=/usr/local/webserver/php/lib/php/extensions/xdebug.so
xdebug.remote_enable=on
; 此地址为IDE所在IP
xdebug.remote_host=xxx.xxx.xxx.xxx
xdebug.remote_port=9000
; 可以是任意Key，这里设定为PHPSTORM
xdebug.idekey=PHPSTORM
```
### 配置IDE

我是用的IDE是PHPStorm，所以以下配置均根据PHPStorm进行，其他如Netbean和Eclipce类似

### 开始DEBUG

设置完成后，在PHPStorm里添加相应的断点，然后用刚配置好的浏览器访问相应页面，首次打开PHPStorm会提示是否接收来自PHP所在服务器的连接。
