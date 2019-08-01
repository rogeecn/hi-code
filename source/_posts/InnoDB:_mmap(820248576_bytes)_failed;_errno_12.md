---
title: 'InnoDB: mmap(820248576 bytes) failed; errno 12'
date: 2018-01-14 00:00:00
tags: ["MySQL"]
abbrlink: mysql-innodb-errno-12
img: ""
comments: false
---

内存空间不足，修改`innodb_buffer_pool_size`或者加大swap分区空间

## 为Linux系统手工添加SWAP空间
在SWAP空间不够用的情况下，如何手工添加SWAP空间？以下的操作都要在root用户下进行：

首先先建立一个分区，采用dd命令比如
```
dd if=/dev/zero of=/home/swap bs=1024 count=512000
```

这样就会创建/home/swap这么一个分区文件。文件的大小是512000个block，一般情况下1个block为1K，所以这里空间是512M。接着再把这个分区变成swap分区。

```
/sbin/mkswap /home/swap
```


## 再接着使用这个swap分区。使其成为有效状态。

```
sbin/swapon /home/swap
```

现在再用free -m命令查看一下内存和swap分区大小，就发现增加了512M的空间了。不过当计算机重启了以后，发现swap还是原来那么大，新的swap没有自动启动，还要手动启动。那我们需要修改`/etc/fstab`文件，增加如下一行
```
/home/swap swap swap defaults 0 0
```

你就会发现你的机器自动启动以后swap空间也增大了。




## mmap
`mmap`可以把磁盘文件的一部分直接映射到内存，这样文件中的位置直接就有对应的内存地址，对文件的读写可以直接用指针来做而不需要read /write 函数。

mmap将一个文件或者其它对象映射进内存。文件被映射到多个页上，如果文件的大小不是所有页的大小之和，最后一个页不被使用的空间将会清零。munmap执行相反的操作，删除特定地址区域的对象映射。 

基于文件的映射，在mmap和munmap执行过程的任何时刻，被映射文件的st_atime可能被更新。如果st_atime字段在前述的情况下没有得到更新，首次对映射区的第一个页索引时会更新该字段的值。用PROT_WRITE 和 MAP_SHARED标志建立起来的文件映射，其st_ctime 和 st_mtime

在对映射区写入之后，但在msync()通过MS_SYNC 和 MS_ASYNC两个标志调用之前会被更新。
