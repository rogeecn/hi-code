---
title: 如何查看一个Windows程序的启动参数
date: 2018-01-03 00:00:00
tags: ["wmic"]
abbrlink: how-to-find-a-windows-program-start-arguments
img: ""
comments: false
---

在windows常使用的一些程序中有些程序是要靠一些启动参数才可以运行成功的， 这里列出一个查看程序启动参数的便捷方法：

`Win+r`打开运行窗口，输入 `wmic`
出现提示后，再输入`process`，就会显示所有进程的命令行信息
