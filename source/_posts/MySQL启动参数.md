---
title: MySQL启动参数
date: 2018-03-14 00:00:00
tags: ["MySQL"]
abbrlink: mysql-startup-param-skip-grant-tables
img: ""
comments: false
---

介绍一个非常有用的mysql启动参数 `--skip-grant-tables`。顾名思义,就是在启动`mysql`时不启动`grant-tables`,授权表。有什么用呢?当然是忘记管理员密码后有用。
## 操作方法:
1、杀掉原来进行着的mysql:
```
rcmysqld stop
```
或者:
```
service mysqld stop
```
或者:
```
kill -TERM mysqld
```
2、以命令行参数启动mysql:
```
/usr/bin/mysqld_safe --skip-grant-tables &
```

3、修改管理员密码:
```
use mysql; 
update user set password=password('yournewpasswordhere') where user='root';
flush privileges; 
exit; 
```
4、杀死mysql,重启mysql

如果你在my.cnf中的有添加"skip-grant-tables",那么任何的帐号用任何的密码(当然也包括空)都可以登录到mysql数据库了。
