---
title: 小米手机root后MIUI系统System目录不可写问题解决
date: 2018-01-31 00:00:00
tags: ["root"]
abbrlink: xiaomi-mobile-after-can-not-write-system-directory
img: ""
comments: false
---

小米手机root后MIUI系统System目录直接写会提示没有权限，需要重新挂载目录进行权限切换，先把手机连接电脑，使用adb连接手机（adb 1.032以上版本），运行下面的命令：

```
adb root
adb disable-verity
adb reboot
adb shell
mount -o rw,remount,rw /system
```

测试下效果，删除自带的搜狗输入法
```
rm -r /system/app/SogouInput
```
