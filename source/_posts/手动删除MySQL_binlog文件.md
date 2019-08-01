---
title: 手动删除MySQL binlog文件
date: 2018-05-22 00:00:00
tags: ["MySQL"]
abbrlink: shou-dong-shan-chu-mysql-binlog-wen-jian
img: ""
comments: false
---

今天 `redis` bg-save进程报错提示磁盘不够用，看了下磁盘发现 `MySQL`的 `bin-log`文件占用了磁盘大部分的空间， 我的 `MySQL` 中 `binlog` 过期天数设置的为 `30` 天，最进写入比较多所以导致 `binlog` 文件数量急增，而 `redis` 数据存储和 `MySQL` 的数据存储在一个磁盘下，所以需要手动删除部分没用的 `binlog` 文件。

有三种解决方法：
1. 关闭mysql主从，关闭binlog；
2. 开启mysql主从，设置expire_logs_days；
3. 手动清除binlog文件，`PURGE MASTER LOGS TO 'MySQL-bin.010'`;



## 操作方法：
### 方案一：关闭mysql主从，关闭binlog

```bash
# vim /etc/my.cnf  //注释掉log-bin,binlog_format
# Replication Master Server (default)
# binary logging is required for replication
# log-bin=mysql-bin
# binary logging format - mixed recommended
# binlog_format=mixed
```
然后重启数据库

### 方案二：重启MySQL,开启MySQL主从，设置expire_logs_days

```bash
# vim /etc/my.cnf  //修改expire_logs_days,x是自动删除的天数，一般将x设置为短点，如10
# expire_logs_days = x  //二进制日志自动删除的天数。默认值为0,表示“没有自动删除”
```
此方法需要重启mysql，附录有关于expire_logs_days的英文说明

当然也可以不重启mysql,开启mysql主从，直接在mysql里设置expire_logs_days
```
> show binary logs;
> show variables like '%log%';
> set global expire_logs_days = 10;
```
 
### 方案三：手动清除binlog文件

```bash
# /usr/local/mysql/bin/mysql -u root -p
> PURGE MASTER LOGS BEFORE DATE_SUB(CURRENT_DATE, INTERVAL 10 DAY);   //删除10天前的MySQL binlog日志,附录2有关于PURGE MASTER LOGS手动删除用法及示例
> show master logs;
```
 
也可以重置master，删除所有binlog文件：

```bash
# /usr/local/mysql/bin/mysql -u root -p
> reset master;  //附录3有清除binlog时，对从mysql的影响说明
```
 
## 附录：
1.expire_logs_days英文说明

> Where X is the number of days you’d like to keep them around. I would recommend 10, but this depends on how busy your MySQL server is and how fast these log files grow. Just make sure it is longer than the slowest slave takes to replicate the data from your master.
> Just a side note: You know that you should do this anyway, but make sure you back up your mysql database. The binary log can be used to recover the database in certain situations; so having a backup ensures that if your database server does crash, you will be able to recover the data.

2.PURGE MASTER LOGS手动删除用法及示例,MASTER和BINARY是同义词
```bash
> PURGE {MASTER | BINARY} LOGS TO 'log_name'
> PURGE {MASTER | BINARY} LOGS BEFORE 'date'
```
删除指定的日志或日期之前的日志索引中的所有二进制日志。这些日志也会从记录在日志索引文件中的清单中被删除MySQL BIN-LOG 日志，这样被给定的日志成为第一个。

### 实例：

```
> PURGE MASTER LOGS TO 'MySQL-bin.010';  //清除MySQL-bin.010日志
> PURGE MASTER LOGS BEFORE '2008-06-22 13:00:00';   //清除2008-06-22 13:00:00前binlog日志
> PURGE MASTER LOGS BEFORE DATE_SUB( NOW( ), INTERVAL 3 DAY);  //清除3天前binlog日志BEFORE，变量的date自变量可以为'YYYY-MM-DD hh:mm:ss'格式。
```
 
3.清除binlog时，对从mysql的影响
如果您有一个活性的从属服务器，该服务器当前正在读取您正在试图删除的日志之一，则本语句不会起作用，而是会失败，并伴随一个错误。不过，如果从属服务器是休止的，并且您碰巧清理了其想要读取的日志之一，则从属服务器启动后不能复制。当从属服务器正在复制时，本语句可以安全运行。您不需要停止它们。
