---
title: NPM 国内镜像使用方法
date: 2018-04-15 00:00:00
tags: ["NodeJS"]
abbrlink: nodejs-npm-module-user-china-mirrors
img: ""
comments: false
---

npm官方站点： http://www.npmjs.org/

本文使用国内镜像地址： http://www.cnpmjs.org/

搜索镜像：https://npm.taobao.org/



## 具体方法：

推荐使用最后一种方法，一劳永逸，前面2钟方法都是临时改变包下载源。

1. 通过 config 配置指向国内镜像源
```
npm config set registry http://registry.cnpmjs.org //配置指向源
npm info express  //下载安装第三方包
```

2. 通过 npm 命令行指定下载源
```
npm --registry http://registry.cnpmjs.org info express
```

3. 在配置文件 ~/.npmrc 文件写入源地址

具体配置文件地址为node安装目录下的npmrc文件，可直接在node安装目录下搜索 npmrc，这时会出现4个文件，选择没有后缀名的进行修改，直接，在最后添加一行 ：
```
registry = http://registry.cnpmjs.org
```

 如果你不像使用国内镜像站点,只需要将 写入 ~/.npmrc 的配置内容删除即可.
