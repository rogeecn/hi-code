---
title: '[转]MySQL权限体系介绍'
date: 2018-01-15 00:00:00
tags: ["MySQL"]
abbrlink: mysql-privilege-produce
img: ""
comments: false
---

## 一、权限体系简介：
MySQL的权限体系在实现上比较简单，相关权限信息主要存储在`mysql.User`、`mysql.db`、`mysql.Host`、`mysql_table_priv`和`mysql.column_priv`几个表中。由于权限信息数据量比较小，而且访问又比较频繁，所以MySQL在启动时就会将所有的权限信息都Load到内存中保存在几个特定的结构中，所以才有了我们手动修改了权限相关的表后，都需要通过执行`FLUSH PRIVILEGES`命令重新加载MySQL的权限信息。我们也可以通过`GRANT`,`REVOKE`或者`DROP USER`命令所做的修改权限后也会同时更新到内存结构中的权限信息。


## 二、权限的赋予与去除
要为某个用户授权可以使用GRANT命令，要去除某个用户现有的权限可以使用REVKOE命令,当给用户授权不仅需要提供用户名，还可以指定通过哪个主机访问，下面提供给简单的列子：

创建一个用户test1只能从本机登录并赋予这个用户拥有test库的查询权限
```
mysql> grant select  on test.* to test1@'localhost' identified by 'test123';
Query OK, 0 rows affected (0.03 sec)
```

创建一个用户test2可以从互联网上任何一台主机登录并赋予这个用户拥有test库的查询权限
```
mysql> grant select  on test.* to test2@'%' identified by 'test234';
Query OK, 0 rows affected (0.02 sec)
```



刷新权限，并查询用户test1的权限
```
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
mysql> show grants for test1@'localhost';
+--------------------------------------------------------------------------------------------------------------+
| Grants for test1@localhost                                                                                   |
+--------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'test1'@'localhost' IDENTIFIED BY PASSWORD '*676243218923905CF94CB52A3C9D3EB30CE8E20D' |
| GRANT SELECT ON `test`.* TO 'test1'@'localhost'                                                              |
+--------------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)
```

删除用户test1的权限
```
mysql> revoke select on test.* from 'test1'@'localhost' identified by 'test123';
Query OK, 0 rows affected (0.00 sec)
```
在此查看用户test1，已经没有权限了。
```
mysql> show grants for test1@'localhost';
+--------------------------------------------------------------------------------------------------------------+
| Grants for test1@localhost                                                                                   |
+--------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'test1'@'localhost' IDENTIFIED BY PASSWORD '*676243218923905CF94CB52A3C9D3EB30CE8E20D' |
+--------------------------------------------------------------------------------------------------------------+
```

三、权限级别
mysql的权限分为5个级别，分别如下：
1、Global Lovel:
Global Lovel的权限控制又称为全局控制权限，所有权限信息u保存在mysql.User 表中，Global Lovel的所有权限都是针对整个mysqld的，对所有mysql数据库下的所有表及所有字段都有效。如果一个权限是以Global Lovel来授予的，则会覆盖其他所有级别的相同权限设置。Global Lovel主要有如下权限：

| 命令 | 版本 | 说明 |
| ------------ | ------------ | ------------ |
| ALTER | ALL | 表结构更改权限 |
| ALTER ROUTINE | 5.0.3 | procedure, function 和 trigger等的变更权限 |
| CREATE | ALL | 数据库，表和索引的创建权限 |
| CREATE ROUTINE | 5.0.3+ | procedure, function 和 trigger等的变更权限 |
| CREATE TEMPORARY TABLES | 4.0.2+ | 零时表的创建权限 |
| CREATE USER | 5.0.3+ | 创建用户的权限 |
| CREATE VIEW | 5.0.1+ | 创建视图的权限 |
| DELETE | ALL | 删除表数据的权限 |
| EXECUTE | 5.0.3+ | procedure, function 和 trigger等的执行权限 |
| FILE | ALL | 执行LOAD DATA INFILE 和 SELECT... INTO FILE 的权限 |
| INDEX | ALL | 在已有表上创建索引的权限 |
| INSERT | ALL | 数据插入权限 |
| LOCK TABLES | 4.0.2+ | 执行LOCK TABLES 命令显示给表加锁的权限 |
| PROCESS | ALL | 执行SHOW PROCESSLIST命令的权限 |
| RELOAD | ALL | 执行FLUSH等让数据库重载LOAD某些对象或者数据命令的权限 |
| REPLCATION SLAVE | 4.0.2+ | 主从复制中SLAVE连接用户所需的复制权限 |
| REPLICATION CLIENT | 4.0.2+ | 执行SHOW MASTER STATUS 和SHOW SLAVE STSTUS命令的权限 |
| SELECT | ALL | 数据查询权限 |
| SHOW DATABASES | 4.0.2+ | 执行SHOW DATABASES的权限 |
| SHUTDOWN | ALL | MySQL Server的shut down 权限 |
| SHOW VIEW | 5.0.1+ | 执行SHOW CREATE VIEW命令查看VIEW创建语句的权限 |
| SUPER | 4.0.2+ | 执行kill线程，CHANGE MASTER,PURGE MASTER LOGS, and SET GLOBAL等命令的权限 |
| UPDATE | ALL | 更新数据库的权限 |
| USAGE | ALL | 新创建用户后不授权时所用到拥有最小的权限 |


要授予Global Lovel权限只需要在执行GRANT命令的时候，用*.*来指定范围是Global即可，如果有多个用户，可以使用逗号分隔开，如下：
```
mysql> grant all on *.* to test3,test4@'localhost' identified by 'test123';
Query OK, 0 rows affected (0.00 sec)
```


## 2、Database Level
Database Level是在Global Level之下，其他三个Level之上的权限级别，其作用域即为所指定数据库中的所有对象，和Database Level比 Database Level主要少了以下几个权限`CREATE USER`,`FILE`,`PROCESS`,`RELOAD`,`REPLICATION CLIENT`,`REPLICATION SLAVE`,` SHOW DATABASES`, `SHUTDOWN`,没有增加任何权限。

要授予Database Level权限，用如下方式实现：

1)、在执行GRANT命令的时候，通过database.* 来指定作用域为整个数据库：或者先创建一个没有权限的用户在使用过GRANT命令来授权。
```
mysql> grant all on test.* to test3,test4@'localhost' identified by 'test123';
Query OK, 0 rows affected (0.00 sec)
```

3、Table Level
Table Level权限可以被Global Level和Database Level权限覆盖，Table Level权限的作用域是授权所指定的表，可以通过如下语句来授权
```
mysql> grant all on test.test1 to wolf@'%' identified by 'wolf@123';
Query OK, 0 rows affected (0.01 sec)
mysql> show grants for wolf@'%';
+-----------------------------------------------------------------------------------------------------+
| Grants for wolf@%                                                                                   |
+-----------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'wolf'@'%' IDENTIFIED BY PASSWORD '*F693761139616215C4AC1A7C23A8B8F5B94704D1' |
| GRANT ALL PRIVILEGES ON `test`.`test1` TO 'wolf'@'%'                                                |
+-----------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)
```
Table Level权限由于作用域仅限于每张表，所以权限种类也比较小，只有如下8个权限，`ALTER`,`CREATE`,`DELETE`,`DROP`,`INDEX`,`INSERT`,`SELECT`,`UPDATE`

4、Column Level
Column Level权限的作用域仅限于某个表的某个列，Column Level同样可以被Database Level，Database Level，Table Level同样的权限覆盖掉，由于Column Level权限和Routine Level权限作用域没有重合部分所以不会被覆盖，Column Level权限仅有`SELECT`,`UPDATE`,`INSERT三种`，通过如下方式赋予权限（需要赋予权限的列名用括号括起来）：

```
Query OK, 0 rows affected (0.01 sec)
mysql> show grants for kelly@'%';
+------------------------------------------------------------------------------------------------------+
| Grants for kelly@%                                                                                   |
+------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'kelly'@'%' IDENTIFIED BY PASSWORD '*30F17FEB599168D8F1BC498525B27B83A13F54E3' |
| GRANT SELECT (name, id) ON `test`.`test` TO 'kelly'@'%'                                              |
+------------------------------------------------------------------------------------------------------+
```

5、Routine Level
Routine Level权限主要只有EXECUTE和ALTER ROUTINE两种，主要针对的对象是procedure 和function两种对象，要赋予Routine Level权限需要指定数据库和相关对象，如下：
```
mysql> grant execute on test.pl to kelly@'%';
Query OK, 0 rows affected (0.01 sec)
```
