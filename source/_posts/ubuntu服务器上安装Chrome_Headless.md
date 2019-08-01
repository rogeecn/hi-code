---
title: ubuntu服务器上安装Chrome Headless
date: 2018-03-23 00:00:00
tags: ["Ubuntu"]
abbrlink: install-chrome-headless-on-ubuntu-server
img: ""
comments: false
---

## 安装chrome
测试环境： Ubuntu 16.04
如果是桌面版的ubuntu，直接到官网下载最新版chrome安装就好。
对于服务器版的ubuntu，只能用命令行安装：
> Install Google Chrome
> https://askubuntu.com/questions/79280/how-to-install-chrome-browser-properly-via-command-line

```
sudo apt-get install libxss1 libappindicator1 libindicator7
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome*.deb  # Might show "errors", fixed by next line
sudo apt-get install -f
```

安装好后可以测试下：


```
google-chrome --headless --remote-debugging-port=9222 https://chromium.org --disable-gpu
```

如果需要远程调试需要绑定`0.0.0.0`，添加参数`--remote-debugging-address=0.0.0.0`
这里是使用headless模式进行远程调试，ubuntu上大多没有gpu，所以--disable-gpu以免报错。
之后使用另一个命令行访问本地的9222端口：
```
curl http://localhost:9222
```
能够看到调试信息应该就是装好了。
