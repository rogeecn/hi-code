---
title: MySQL启动slave时报错Slave failed to initialize relay log info structure from the repository
date: 2018-05-18 00:00:00
tags: ["MySQL"]
abbrlink: mysql-qi-dong-slave-shi-bao-cuo-slave-failed-to-initialize-relay-log-info-structure-from-the-repository
img: ""
comments: false
---

## 症状：

MySQL主从复制，启动slave时，出现下面报错： 
```
mysql> start slave; 
ERROR 1872 (HY000): Slave failed to initialize relay log info structure from the repository
```



## 解决办法：
查看日志，可以看到报错，原来是找不到 `./server246-relay-bin.index` 文件，找到原因所在了，由于我使用的是冷备份文件恢复的实例，在mysql库中的slave_relay_log_info表中依然保留之前relay_log的信息，所以导致启动slave报错。 
mysql提供了工具用来删除记录：`reset slave`； 

reset slave 执行候做了这样几件事： 
1. 删除slave_master_info ，slave_relay_log_info两个表中数据； 
2. 删除所有relay log文件，并重新创建新的relay log文件； 
3. 不会改变 `gtid_executed` 或者 `gtid_purged` 的值

```
mysql> reset slave;
Query OK, 0 rows affected (0.01 sec)

mysql> change master to ......

mysql> start slave;
Query OK, 0 rows affected (0.00 sec)
```
这样slave 就可以启动了。
