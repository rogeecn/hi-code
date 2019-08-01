---
title: 解压缩rpm和deb包
date: 2018-03-16 00:00:00
tags: ["Ubuntu"]
abbrlink: unpack-rpm-or-deb-package
img: ""
comments: false
---

## 1.解压rpm包
rpm2cpio xx.rpm|cpio -idmv

## 2.解压deb包
```
dpkg-deb --fsys-tarfile ***.deb | tar xvf -
```
如果没有`dpkg-deb`命令，可使用：
```
ar -x ***.deb
```
解压完会出现两个tar包，使用tar解包即可。



## 3.查找文件来自于哪个包
### ubuntu: 
- `dlocate foo` - 在已安装的包中搜索“foo”的文件。对于回答“这个文件来源于哪个包”这个问题，是非常实用的。dlocate是一个软件包，必须安装它才能使用本命令。
- `dpkg -S foo` - 和上面的命令一样，但相比更慢一些。他只能在Debian或Ubuntu系统下运行。另外，不需要安装dlocate包。 

### Redhat:
```
rpm -qf foo
```
