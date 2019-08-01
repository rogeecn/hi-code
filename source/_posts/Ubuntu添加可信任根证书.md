---
title: Ubuntu添加可信任根证书
date: 2018-03-13 00:00:00
tags: ["Ubuntu"]
abbrlink: ubuntu-add-ca-certificates
img: ""
comments: false
---

## 添加
Ubuntu下添加根证书非常简单, 只要将证书(扩展名为crt)复制到**/usr/local/share/ca-certificates**文件夹然后运行`update-ca-certificates`即可
```
[yaxin@ubox ~]$sudo cp xinmu.crt /usr/local/share/ca-certificates
[yaxin@ubox ~]$sudo update-ca-certificates
Updating certificates in /etc/ssl/certs... 1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d....done.
```

## 删除
直接将**/usr/local/share/ca-certificates**对应的证书删除，然后执行update-ca-certificates
```
[yaxin@ubox ~]$sudo rm -f /usr/local/share/ca-certificates/xinmu.crt
[yaxin@ubox ~]$sudo update-ca-certificates
Updating certificates in /etc/ssl/certs... 0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d....done.
```
注意: 这时候并不会提示`1 removed`, 但证书是已经被删除了的

## 原理
其实`update-ca-certificates`是一个shell脚本, 使用`which`找出`update-ca-certificates`的绝对路径，然后打开就可以查看其源码
```
[yaxin@ubox ~]$which update-ca-certificates
/usr/sbin/update-ca-certificates
[yaxin@ubox ~]$file /usr/sbin/update-ca-certificates
/usr/sbin/update-ca-certificates: POSIX shell script, ASCII text executable
```
通过阅读源码可以看出, `update-ca-certificates`命令的本质其实是将PEM格式的根证书内容附加到*/etc/ssl/certs/ca-certificates.crt*, 而*/etc/ssl/certs/ca-certificates.crt*中本身就包含了系统自带的各种可信根证书.
