---
title: Docker使用OverlayFS做Storage Driver需要注意的问题
date: 2018-02-14 00:00:00
tags: ["Docker"]
abbrlink: docker-needs-to-pay-attention-to-the-use-of-overlayfs-to-do-storage-driver
img: ""
comments: false
---

我们部署容器服务时，如果Node Host使用Centos7.2，默认的文件系统使用xfs（这个特性从Centos7开始引入）。而我们打算采用的Docker Engine是17.04，这个默认的storage driver为overlay。在Centos7.2上使用overlay需要注意的几点：
Host的`SElinux`必须开启
创建`xfs`文件系需要加上`-n ftype=1`
...
更多内容在[Centos7.2 Release Notes CHARTER 21. FILE SYSTEMS](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/7.2_release_notes/technology-preview-file_systems "Centos7.2 Release Notes CHARTER 21. FILE SYSTEMS"),对于第二条如果不加这个参数，就会出现奇奇怪怪的问题，比如删除目录会报“Directory not empty”错误，启动容器报“mkdir ...-init/merged/dev/shm: invalid argument”错误。
接下来主要测试xfs不开启ftype对overlay的影响。



测试环境
OS：Centos7.2
Docker：17.04.0-ce
测试步骤
查看环境信息
```bash
[root@k8s ~]# docker info
Containers: 7
 Running: 5
 Paused: 0
 Stopped: 2
Images: 19
Server Version: 17.04.0-ce
Storage Driver: overlay
 Backing Filesystem: xfs
 Supports d_type: false
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Init Binary:
containerd version: 422e31ce907fd9c3833a38d7b8fdd023e5a76e73
runc version: 9c2d8d184e5da67c95d601382adf14862e4f2228
init version: 949e6fa
Security Options:
 seccomp
  Profile: default
Kernel Version: 3.10.0-327.el7.x86_64
Operating System: CentOS Linux 7 (Core)
OSType: linux
Architecture: x86_64
CPUs: 8
Total Memory: 7.796GiB
Name: k8s
ID: HXL3:J5JB:R4CQ:A6LA:2MYJ:62S6:OB2G:CO6L:UEY3:MT6R:5HXP:RLCQ
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
Experimental: false
Insecure Registries:
 10.187.161.50
 127.0.0.0/8
Live Restore Enabled: false
 
WARNING: overlay: the backing xfs filesystem is formatted without d_type support, which leads to incorrect behavior.
         Reformat the filesystem with ftype=1 to enable d_type support.
         Running without d_type support will not be supported in future releases.
```

如上，如果xfs不开启ftype，docker会提示warning信息。下面测试再容器中可能会出现的错误，使用[test_xfs_without_d_type.sh](https://gist.github.com/latelan/c138e2e63c1540a11956aa0022b7275b "test_xfs_without_d_type.sh")测试

```bash
$ fallocate -l 2GB testvolume
$ device=$(sudo losetup -f --show ./testvolume)
$ sudo mkfs -t xfs -n ftype=0 -f $device
$ sudo sh test_xfs_without_d_type.sh $device overlay ubuntu:latest rm -rf /var/lib/*
rm: cannot remove '/var/lib/apt/keyrings': Directory not empty
rm: cannot remove '/var/lib/apt/mirrors': Directory not empty
rm: cannot remove '/var/lib/dpkg/alternatives': Directory not empty
rm: cannot remove '/var/lib/dpkg/info': Directory not empty
rm: cannot remove '/var/lib/dpkg/triggers': Directory not empty
rm: cannot remove '/var/lib/pam': Directory not empty
rm: cannot remove '/var/lib/systemd/catalog': Directory not empty
rm: cannot remove '/var/lib/systemd/deb-systemd-helper-enabled/timers.target.wants': Directory not empty
 
 
开启ftype再测试
$ sudo mkfs -t xfs -n ftype=1 -f $device
$ sudo sh test_xfs_without_d_type.sh $device overlay ubuntu:latest rm -rf /var/lib/*
```

还有的错误在[这里](https://github.com/moby/moby/issues/22937 "这里")
总结
如果我们使用Centos7.2+xfs+overlay，注意# mkfs -t xfs -n ftype=1 /PATH/TO/DEVICE


参考：
- https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/7.2_release_notes/technology-preview-file_systems
- https://docs.docker.com/engine/userguide/storagedriver/overlayfs-driver/
- http://man7.org/linux/man-pages/man8/mkfs.xfs.8.html
- https://linuxer.pro/2017/03/what-is-d_type-and-why-docker-overlayfs-need-it/
- https://github.com/sameersbn/docker-redmine/issues/260
