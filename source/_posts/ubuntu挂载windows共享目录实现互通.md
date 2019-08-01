---
title: ubuntu挂载windows共享目录实现互通
date: 2018-03-01 00:00:00
tags: ["windows"]
abbrlink: ubuntu-mont-windows-share-directory
img: ""
comments: false
---

1. 首先WINDOWS需要创建一个共享目录。这里不再表述。
2. ubuntu 创建挂载点目录`/mnt/projects`
3. /etc/fstab 中写入下面命令
```
//server.ip/projects /mnt/projects cifs username=xxx,passwd=xxx,vers=2.0,uid=rogee,gid=rogee 0 0
```

这里有需要注意的几点
- 这里vers=2.0 加上，否则在新内核中会有问题
- uid,gid 在非root用户共享时需要指定，否则没有写入权限提示`Permission Denied`
- cifs 全称为`common internet file system`通过网络文件系统，如果系统`/mount/mount.cifs`不存在需要先安装`sudo apt-get install cifs-utils`

4. 运行命令对网络目录进行挂载 `sudo mount -a`
5. 查看是否挂载成功`ls /mnt/projects`


相关问题链接：
- [https://askubuntu.com/questions/915549/16-04-cifs-host-is-down-but-they-are-not](https://askubuntu.com/questions/915549/16-04-cifs-host-is-down-but-they-are-not)
- [https://askubuntu.com/questions/393563/how-to-change-permissions-on-mounted-windows-share](https://askubuntu.com/questions/393563/how-to-change-permissions-on-mounted-windows-share)
