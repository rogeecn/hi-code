---
title: 卸载(删除)mac下自带的php
date: 2018-04-14 00:00:00
tags: ["php"]
abbrlink: remove-system-php-in-macos
img: ""
comments: false
---

## 关闭 Rootless
重启按住 Command+R，进入恢复模式，顶部菜单“utility”，打开Terminal。

```
csrutil disable
```

重启即可。如果要恢复默认，那么
```
csrutil enable
```



删除相关PHPyywr
```
/private/etc/               sudo rm -rf php-fpm.conf.default php.ini php.ini.default
/usr/bin/               sudo rm -rf php php-config phpdoc phpize
/usr/include                sudo rm -rf php
/usr/lib                sudo rm -rf php
/usr/sbin               sudo rm -rf php-fpm
/usr/share              sudo rm -rf php
/usr/share/man/man1         sudo rm -rf php-config.1 php.1 phpize.1
/usr/share/man/man8         sudo rm -rf php-fpm.8

```
