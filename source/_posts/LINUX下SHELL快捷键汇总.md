---
title: LINUX下SHELL快捷键汇总
date: 2018-01-12 00:00:00
tags: ["Linux"]
abbrlink: shortcut-for-linux-shell-command
img: ""
comments: false
---

在linux系统下，虽然已经习惯敲打命令行的工作方式，但是效率不是十分高，一旦你熟悉以下这些快捷键之后，相信你的工作效率会提高N倍，下面就来体验下吧：



1. `ctrl +a`  切换到命令行开始
这个操作跟Home实现的结果一样的，但Home在某些Unix环境下无法使用，便可以使用这个组合；在Linux下的vim，这个也是有效的；
2. `ctrl+e`   切换到命令行末尾
这个操作跟END实现的结果一样的，但End键在某些Unix环境下无法使用，便可以使用这个组合；在Linux下的vim，这个也是有效的
3. `ctrl+l`    清除屏幕内容，效果等同于clear
4. `ctrl+u`  清除剪切光标之前的内容。这个命令十分有效，相信遇到输错命令，一个个删除字符的时候那个痛苦啊， 现在一个快捷键就可以搞定了，嘿嘿
5. `ctrl+k`  剪切清除光标之后的内容
6. `ctrl+y`  粘贴刚才删除的字符
7. `Ctrl + r` 在历史命令中查找 （这个非常好用，输入关键字就调出以前的命令了）
8. `ctrl +c`  终止命令
9. `ctrl + d` 退出shell，logout
10.`ctrl + z` 转入后台运行
11.`!$` 显示系统最近的一条参数

这个命令十分有意思，比如我刚刚还在看一个文件的内容，比如`cat /etc/sysconfig/network-scripts/ifcfg-eth0`,现在我想编辑这个文件的内容，我们一般是先按 ↑  键来找到之前的命令，然后用HOME键或者`CTRL +A`，然后把`cat`修改成`vi`，但是现在介绍下，你可以直接这样，`vi !$`,这样就可以编辑了。
