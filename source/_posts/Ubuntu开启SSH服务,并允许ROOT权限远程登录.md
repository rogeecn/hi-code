---
title: Ubuntu开启SSH服务,并允许ROOT权限远程登录
date: 2018-05-09 00:00:00
tags: ["Ubuntu"]
abbrlink: ubuntu-kai-qi-ssh-fu-wu-bing-yun-xu-root-quan-xian-yuan-cheng-deng-lu
img: ""
comments: false
---

服务器配完ubuntu系统以及LNMP环境以后，想用WINSCP远程登录，就需要开启SSH服务才能支持。

 SSH服务分为客户端和服务器。顾名思义，我想用WINSCP远程登录Ubuntu服务器，所以需要安装SSH server。

OK，下面介绍如何开启SSH服务。



## SSH开启

### 一、检查是否开启SSH服务 

因为Ubuntu默认是不安装SSH服务的，所以在安装之前可以查看目前系统是否安装，通过以下命令：

```
 ps -e|grep ssh 
 ```

输出的结果ssh-agent表示ssh-client启动，sshd表示ssh-server启动。我们是需要安装服务端所以应该看是否有sshd，如果没有则说明没有安装。

### 二、安装SSH服务

```
 sudo apt-get install openssh-client 客户端

 sudo apt-get install openssh-server 服务器
````

或者
```
apt-get install ssh
```

### 三、启动SSH服务 

```
 sudo /etc/init.d/ssh start
 ```

### 四、修改SSH配置文件 

可以通过SSH配置文件更改包括端口、是否允许root登录等设置，配置文件位置：
```
 /etc/ssh/sshd_config
 ```

 默认是不允许root远程登录的，可以再配置文件开启。
```
 sudo vi /etc/ssh/sshd_config
 ```
 找到 `PermitRootLogin without-password` 修改为 `PermitRootLogin yes`
 
###  五、重启SSH服务
```
 service ssh restart
``` 
 即可通过winscp 、putty使用ROOT权限远程登录。
 启用root用户：
 ```
 sudo passwd root      //修改密码后就启用了。
``` 

 客户端如果是ubuntu的话，则已经安装好ssh client,可以用下面的命令连接远程服务器。
```
$ ssh xxx.xxx.xxx.xxx
```

 

## 简单介绍下SSH：

SSH：是一种安全通道协议，主要用来实现字符界面的远程登录，远程复制等功能(使用TCP的22号端口)。SSH协议对通信双方的数据传输进行了加密处理，其中包括用户登录时输入的用户口令。

在RHEL 5系统中使用的是OpenSSH服务器由openssh，openssh-server等软件包提供的(默认已经安装)，并以将sshd添加为标准的系统服务。

## SSH提供一下两种方式的登录验证：

1. 密码验证：以服务器中本地系统用户的登录名称，密码进行验证。
2. 秘钥对验证：要求提供相匹配的秘钥信息才能通过验证。通常先在客户机中创建一对秘钥文件(公钥和私钥)，然后将公钥文件放到服务器中的指定位置。

注意：当密码验证和私钥验证都启用时，服务器将优先使用秘钥验证。

## SSH的配置文件：

sshd服务的配置文件默认在/etc/ssh/sshd_config，正确调整相关配置项，可以进一步提高sshd远程登录的安全性。

配置文件的内容可以分为以下三个部分：

### 1、常见SSH服务器监听的选项如下：

- Port 22                    //监听的端口为22
- Protocol 2                //使用SSH V2协议
- ListenAdderss 0.0.0.0    //监听的地址为所有地址
- UseDNS no                //禁止DNS反向解析

### 2、常见用户登录控制选项如下：

- PermitRootLogin no            //禁止root用户登录
- PermitEmptyPasswords no        //禁止空密码用户登录
- LoginGraceTime 2m            //登录验证时间为2分钟
- MaxAuthTries 6                //最大重试次数为6
- AllowUsers user            //只允许user用户登录，与DenyUsers选项相反

### 3、常见登录验证方式如下：

- PasswordAuthentication yes                //启用密码验证
- PubkeyAuthentication yes                    //启用秘钥验证
- AuthorsizedKeysFile .ssh/authorized_keys    //指定公钥数据库文件
