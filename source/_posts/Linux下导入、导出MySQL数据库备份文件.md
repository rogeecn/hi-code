---
title: Linux下导入、导出MySQL数据库备份文件
date: 2018-03-31 00:00:00
tags: ["MySQL"]
abbrlink: linux-dump-import-mysql-databases
img: ""
comments: false
---

## 一、导出数据库用mysqldump命令
（注意mysql的安装路径，即此命令的路径）：
### 1、导出数据和表结构：
```
mysqldump -u用户名 -p密码 数据库名 > 数据库名.sql
```

敲回车后会提示输入密码

### 2、只导出表结构
```
mysqldump -u用户名 -p密码 -d 数据库名 > 数据库名.sql
```
> 注：/usr/local/mysql/bin/  --->  mysql的data目录




## 二、导入数据库
### 1、首先建空数据库
```
mysql>create database abc;
```

### 2、导入数据库
#### 方法一：
（1）选择数据库
```
mysql>use abc;
```
（2）设置数据库编码
```
mysql>set names utf8;
```
（3）导入数据（注意sql文件的路径）
```
mysql>source /home/abc/abc.sql;
```
### 方法二：
```
mysql -u用户名 -p密码 数据库名 < 数据库名.sql
```
建议使用第二种方法导入。
