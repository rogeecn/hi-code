---
title: MySQL字符集合配置UTF8
date: 2018-03-24 00:00:00
tags: ["MySQL"]
abbrlink: mysql-set-default-character-to-utf8
img: ""
comments: false
---

```
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8


[mysqld]
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server = utf8
```
