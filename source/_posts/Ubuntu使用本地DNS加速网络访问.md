---
title: Ubuntu使用本地DNS加速网络访问
date: 2018-03-29 00:00:00
tags: ["Ubuntu"]
abbrlink: ubuntu-use-local-dns-to-speedup-network
img: ""
comments: false
---

安装 dnsmasq
```
sudo apt-get install dnsmasq
```
## 配置 dnsmasq
看dnsmasq帮助
```
man dnsmasq
```
其中有这么一段描述： 
> In order to configure dnsmasq to act as cache for the host on which it is running, put "nameserver 127.0.0.1" in /etc/resolv.conf to force local processes to send queries to dnsmasq. Then either specify the upstream servers directly to dnsmasq using --server options or put their addresses real in another file, say /etc/resolv.dnsmasq and run dnsmasq with the -r /etc/resolv.dnsmasq option.

大意是如果想让dnsmasq作为dns缓存，需要将`nameserver 127.0.0.1`放到`/etc/resolv.conf`文件中，通常是第一条非注释语句，然后将真正的dns服务器信息放到另外一个文件中，如`/etc/resolv.dnsmasq`，最后执行命令：
```
dnsmasq -r /etc/resolv.dnsmasq
```


## 第一步
按照帮助文档的提示，需要修改`/etc/resolv.conf`文件。 可以手动修改，如使用vi，可以将原有的内容全部注释，然后在第一行写上
```
nameserver 127.0.0.1；
```
也可以使用ubuntu的网络管理小程序“Network Manager”在桌面右上角有一个它的图标，右键点击该图标，选择“编辑连接”，选择你所使用的连接，点击编辑，在“IPv4设置”标签的“DNS服务器”输入框中，把原有的DNS服务器删除，输入
```
127.0.0.1。
```
## 第二步
在/etc目录下新建`resolv.dnsmasq`文件。文件的内容为DNS服务器的地址，是真正的DNS服务器，如我的文件内容是：
```
nameserver 210.47.0.1
nameserver 202.98.5.68
```
## 第三步
可以不按帮助文档所说的执行“dnsmasq -r /etc/resolv.dnsmasq”命令，如果这样，岂不是每次都得在命令行里输入，非常麻烦，当然，可以考虑把这个命令写入“/etc/rc.local”文件中，让系统每次启动时帮你运行。 我所使用的方法是编辑“/etc/dnsmasq.conf”文件。找到下面这一项
```
#resolv-file=
```
用下面的一条语句替换
```
resolv-file=/etc/resolv.dnsmasq
```
其实也就是执行dnsmasq命令中-r参数后面的内容。

编辑 `/etc/dhcp3/dhclient.conf`
找到下面这一项
```
#prepend domain-name-servers 127.0.0.1;
```
将前面的“#”删除。这么做的目的是为了在使用自动连接时，能在/etc/resolv.conf文件的第一行添加上“nameserver 127.0.0.1”，这样，dns缓存依然有效

编辑 `/etc/ppp/peers/dsl-provider`
可能有的系统没有`/etc/ppp/peers/dsl-provider`文件，而是`/etc/ppp/peers/provider`文件，找到下面这一项
```
usepeerdns
```
在前面增加`#`，也就是把这条语句注释掉。以防resolv.conf的设置被pppoe复盖。

对于12.04版本 由于该版本已经安装dnsmasq-base，则必须先修改`/etc/NetworkManager/NetworkManager.conf`文件，注释`dns=dnsmasq` 修改`/etc/default/dnsmasq`文件，取消`IGNORE_RESOLVCONF=yes`注释

测试
重启服务：
```
sudo /etc/init.d/dnsmasq restart
```
或者
```
sudo service dnsmasq restart
```

测试，执行两次就能看出查询时间的差异了：

dig g.cn
