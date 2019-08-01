---
title: Ubuntu 16.04 配置Docker启动参数
date: 2018-04-03 00:00:00
tags: ["Ubuntu"]
abbrlink: ubuntu-set-docker-startup-params
img: ""
comments: false
---

### 编辑服务

```
sudo systemctl edit docker.service
```
设置启动参数

```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375
```

### 重载服务
```
sudo systemctl daemon-reload
sudo systemctl restart docker.service
sudo ps aux |grep dockerd
```
