---
title: 使用Cow搭建代理服务实现go get获取被墙包
date: 2018-05-16 00:00:00
tags: ["Cow"]
abbrlink: shi-yong-cow-da-jian-dai-li-fu-wu-shi-xian-go-get-huo-qu-bei-qiang-bao
img: ""
comments: false
---

使用go get经常会碰到`go get`无法访问的问题，使用VPN就能很好的解决这个方法，但是VPN经常被封啊。  现在基本都是用`shadowsocks`来翻墙，但是`shadowsocks`是基于 `socks5` 协议的，`go get` 则是使用 `http` 协议进行网络访问，因此开了 `shadowsocks`， `go get` 依旧不通。如果我们能把 `socks5` 代理转为 `http` 代理，那就可以访问了。


好消息是大多数 `gnu unix` 程序都会访问一个环境变量 `http_proxy` ，比如 `curl` 在执行时会先访问一下 `http_proxy` ， 如果有这个变量的话，会使用这个变量给出的地址作为代理。这个方式支持 http、https、ftp 和 rsync 协议，使用http协议的npm，go get这些程序当然可以愉快的翻墙了。
总结一下我们的使用方式：
```
go get (使用 http) -> http proxy (cow 提供) -> socks5代理（shadowsocks 提供）-> 真正的地址
```



内牛满面， 下一个包这么曲折，心疼自己。
本文认为你已经搭建好了Go的基本开发环境，如果你的Go开发环境还没搭建好，可以参考许瘦子来世写的文章Go开发：Mac上安装Go环境和VS Code
环境
- imac， 10.13.2
- cow

`cow` 据说是 `shadowsocks-go` 作者的另一个开发项目，根据项目介绍简单的配置，就可以在本机启动一个http代理，以`shadowsocks`为二级代理。


## 安装
根据中文官方文档安装 cow

OS X, Linux (x86, ARM): 执行以下命令（也可用于更新）
```
curl -L git.io/cow | bash
```
环境变量 `COW_INSTALLDIR` 可以指定安装的路径，若该环境变量不是目录则询问用户



## 配置
在配置文件 `$HOME/.cow/rc` 中添加两行配置， 分别是 `cow` 的监听和代理

```
listen = http://127.0.0.1:7777  #默认已添加
proxy = socks5://127.0.0.1:1080
```

然后设置环境变量
```
export http_proxy=http://127.0.0.1:7777
export https_proxy=http://127.0.0.1:7777

source ~/.profile
```
查看变量是否生效 `echo $http_proxy`


## 启动cow

```
cow &
```

这里有两个问题需要注意：

- 新版的Mac OS X 有SIP，需要关闭SIP后才能使用cow
- 直接下载执行可能遇到错误：`fatal error: MSpanList_Insert`可以尝试通过源码编译得到可执行文件



## 访问
试一试，是不是可以愉快的go get 了

```
curl https://google.com/
go get -u -v sourcegraph.com/sqs/goreturns
```

还有好多人在命令行使用 `proxychains`，也宣称支持http/https, 但是实际使用`go get` 会报错
```
proxychains4 go get -u -v sourcegraph.com/sqs/goreturns
[proxychains] config file found: /usr/local/etc/proxychains.conf
[proxychains] preloading /usr/local/Cellar/proxychains-ng/4.12_1/lib/libproxychains4.dylib
[proxychains] DLL init: proxychains-ng 4.12
package sourcegraph.com/sqs/goreturns: parse [proxychains] DLL init: proxychains-ng 4.12
https://github.com/sqs/goreturns: first path segment in URL cannot contain colon
```
