---
title: Ubunt 16.04强制刷新DNS缓存
date: 2018-05-13 00:00:00
tags: ["Ubuntu"]
abbrlink: ubunt-16-04-qiang-zhi-shua-xin-dns-huan-cun
img: ""
comments: false
---

## 方法一

运行下面命令进行刷新
```
sudo systemd-resolve --flush-caches
```



如果需要验证是否刷新成功，运行下面的命令
```
sudo systemd-resolve --statistics
```

系统输出样例

```
Cache
  Current Cache Size: 0
          Cache Hits: 101
        Cache Misses: 256
```

## 方法二
直接重启服务
```
systemctl restart systemd-resolved.service
```


方法来源：[How can I flush the DNS on Ubuntu 17.04?](https://askubuntu.com/questions/906476/how-can-i-flush-the-dns-on-ubuntu-17-04 "How can I flush the DNS on Ubuntu 17.04?")
