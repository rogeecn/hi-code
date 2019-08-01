---
title: MySQL 5.7 修改账号密码
date: 2018-04-20 00:00:00
tags: ["MySQL"]
abbrlink: mysql-57-reset-user-password
img: ""
comments: false
---

为了提高安全性 MySQL5.7中user表的password字段已被取消，取而代之的事 authentication_string 字段，当然我们更改用户密码也不可以用原来的修改user表来实现了。下面简绍几种MySQL5.7下修改root密码的方法（其他用户也大同小异）。



## 方法一：
```
mysql> update mysql.user set authentication_string=password('123qwe') where user='root' and Host = 'localhost';
```

## 方法二：
```
mysql> alter user 'root'@'localhost' identified by '123';
```
## 方法三：
```
mysql> set password for 'root'@'localhost'=password('123');
```
记得最后要刷新权限
```
mysql> flush privileges;
```
