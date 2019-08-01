---
title: MySQL手动重置root密码
date: 2018-01-06 00:00:00
tags: ["MySQL"]
abbrlink: how-to-reset-mysql-root-password
img: ""
comments: false
---

1. 在`my.ini`的`[mysqld]`字段加入：`skip-grant-tables`
2. 重启mysql服务，这时的mysql不需要密码即可登录数据库,然后进入mysql运行如下命令：

```
mysql> use mysql;
mysql> updata user set password=password('新密码') WHERE User='root';
mysql> flush privileges;
```
运行之后最后去掉`my.ini`中的`skip-grant-tables`，重启mysql即可。
