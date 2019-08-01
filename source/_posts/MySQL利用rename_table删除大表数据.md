---
title: MySQL利用rename table删除大表数据
date: 2018-02-21 00:00:00
tags: ["MySQL"]
abbrlink: mysql-fast-delete-big-table-rename-table
img: ""
comments: false
---

今天有人提了一个问题

> 一个表有1亿6000万的数据，有一个自增ID。最大值就是1亿6000万，需要删除大于250万以后的数据，有什么办法可以快速删除？

当时看了一眼数据吓尿了，这么大的数据要删除到什么时候啊，最要命的锁表肿么办
delete是不行了，加索引也别想。mysql上delete加low_priorty,quick,ignore估计也帮助不大
看到mysql文档有一种解决方案：http://dev.mysql.com/doc/refman/5.0/en/delete.html



> If you are deleting many rows from a large table, you may exceed the lock table size for an InnoDB table. To avoid this problem, or simply to minimize the time that the table remains locked, the following strategy (which does not use DELETE at all) might be helpful:
> Select the rows not to be deleted into an empty table that has the same structure as the original table:
> INSERT INTO t_copy SELECT * FROM t WHERE ... ;
> Use RENAME TABLE to atomically move the original table out of the way and rename the copy to the original name:
> RENAME TABLE t TO t_old, t_copy TO t;
> Drop the original table:
> DROP TABLE t_old;

E文不好，简单的翻译下：
删除达标上的多行数据时，innodb会超出lock table size的限制，最小化的减少锁表的时间的方案是：
1. 选择不需要删除的数据，并把它们存在一张相同结构的空表里
2. 重命名原始表，并给新表命名为原始表的原始表名
3. 删掉原始表

 总结一下就是，当时删除大表的一部分数据时可以使用 `建立新表-拷贝数据-删除旧表-重命名`的方法。
 
 操作步骤：
 
 查看原表创建语句，创建新表, 这里我使用的表名为`message`

```bash

mysql> show create table message\G;
*************************** 1. row ***************************
       Table: message
Create Table: CREATE TABLE `message` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `message_id` varchar(255) CHARACTER SET utf8 COLLATE utf8_unicode_ci NOT NULL,
  `account_id` int(11) NOT NULL,
  `from_user` int(11) NOT NULL,
  `to_user` int(11) NOT NULL,
  `type` int(11) NOT NULL,
  `content` text NOT NULL,
  `created_at` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=372214 DEFAULT CHARSET=utf8mb4
1 row in set (0.00 sec)

ERROR:
No query specified
```

新表可以把主键置0的话把创建语句的`AUTO_INCREMENT=xxx`去掉,如下, 创建`message_copy`表：
```bash

CREATE TABLE `message_copy` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `message_id` varchar(255) CHARACTER SET utf8 COLLATE utf8_unicode_ci NOT NULL,
  `account_id` int(11) NOT NULL,
  `from_user` int(11) NOT NULL,
  `to_user` int(11) NOT NULL,
  `type` int(11) NOT NULL,
  `content` text NOT NULL,
  `created_at` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
```
使用`show tables`命令已经可以看到，新的copy表已经创建成功了。
如果需要复制部分数据执行下面的命令, where条件语句可以使用自己的插入条件：
```
insert into message_copy select * from message where id > 1000
```

交换两个表的数据，删除老表：
```
RENAME TABLE  message TO message_copy, message_copy TO message;
DROP TABLE message_copy;
```

DONE!
