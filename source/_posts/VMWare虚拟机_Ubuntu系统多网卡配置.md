---
title: VMWare虚拟机 Ubuntu系统多网卡配置
date: 2018-03-12 00:00:00
tags: ["VMWare"]
abbrlink: vmware-ubuntu-multiple-network-interfaces
img: ""
comments: false
---

## 实现：

Ubuntu使用原有的“网卡0”通过主机共享的“本地连接”连接上Internet，再新建一个虚拟网卡“网卡1”，用于Ubuntu虚拟机与Windows主机通信，且具有固定IP地址。


## 各网卡对应关系:
|     -    |          Windows       |    VMWare     | Ubuntu  |    作用
| ------------ | ------------ | ------------ | ------------ | ------------ |
|虚拟网卡1 | VMware Network Adapter | VMnet1 VMnet1 | eth0    | 连接Internet
|虚拟网卡2 | VMware Network Adapter | VMnet8 VMnet8 | eth1    | 主机与虚拟机通信

## 实施过程:
1. 虚拟机添加一块新网卡
2. 连接UBUNTU SSH 打开“interfaces”
```
root@UbuntuLTS:~# vim/etc/network/interfaces
```



编辑文件内容

```
auto lo
iface lo inet loopback


# 自动获取IP
# iface eth0 inet dhcp
# auto eth0


# 指定IP
auto eth0
iface eth0 inet static
address 192.168.137.218
netmask 255.255.255.0
gateway 192.168.137.1
network 192.168.137.0
broadcast 192.168.137.255


#设置eth1
auto eth1
iface eth1 inet static
address 192.168.1.218
netmask 255.255.255.0
gateway 192.168.1.1
network 192.168.1.0
broadcast 192.168.1.255


#We must specify dns-nameserver here
#in order to get internet access from host
dns-nameservers 192.168.137.1
```

3. 保存上述更改，退出
4. 重启网络
```
systemctl restart networking.service
```
