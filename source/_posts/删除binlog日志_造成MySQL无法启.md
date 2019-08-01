---
title: 删除binlog日志 造成MySQL无法启
date: 2018-05-17 00:00:00
tags: ["MySQL"]
abbrlink: shan-chu-binlog-ri-zhi-zao-cheng-mysql-wu-fa-qi
img: ""
comments: false
---

在操作系统上直接删除binlog日志（`rm binlog`）造成MySQL无法启动

```
[ERROR] Failed to open log [ERROR] Could not open log file [ERROR] Can’t init tc log [ERROR] Aborting
```

### 问题描述：
MySQL数据库所显示空间紧张，其中binlog日志占用了很多空间，脑袋一热就把所有binlog日志都给删除了。然后重启MySQL，结果启动不起来。

MySQL重启操作，关闭正常，启动却报如下错误，无法启动。




```
170120  9:54:26 InnoDB: 5.5.48 started; log sequence number 88052315056
/usr/local/mysql/bin/mysqld: File './mybinlog.000014' not found (Errcode: 2)
170120  9:54:26 [ERROR] Failed to open log (file './mybinlog.000014', errno 2)
170120  9:54:26 [ERROR] Could not open log file
170120  9:54:26 [ERROR] Can't init tc log
170120  9:54:26 [ERROR] Aborting
 
170120  9:54:26  InnoDB: Starting shutdown...
170120  9:54:30  InnoDB: Shutdown completed; log sequence number 88052315056
170120  9:54:30 [Note] /usr/local/mysql/bin/mysqld: Shutdown complete
 
170120 09:54:30 mysqld_safe mysqld from pid file /data/mysql/mysql3306/logs/mysql.pid ended
```
### 问题分析：
MySQL启动服务，根据 mysql-bin.index 文件的记录，检查各个binlog文件是否存在，如果不存在则报出上面的错误。

### 解决办法：
将已经删除的binlog条目从 `mysql-bin.index` 文件中删除。MySQL顺利启动。

### 杜绝以上问题的措施：
1. 绝对不能从操作系统上直接删除binlog日志，如rm binlog。
2. 在MySQL中，使用PURGE BINARY LOGS命令删除binlog日志，才是最安全的办法。命令如下，

```
PURGE BINARY LOGS BEFORE '2017-01-20 00:00:00';
```
