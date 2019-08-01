---
title: 使用sshpass直接登陆SSH
date: 2018-03-18 00:00:00
tags: ["SSH"]
abbrlink: use-sshpass-login-sshd
img: ""
comments: false
---

```
sshpass -p 'password' ssh -o StrictHostKeyChecking=no -p 22 root@server.ip
```
命令如上。

- `sshpass -p`录入密码
- `-o StrictHostKeyChecking=no` 忽略`fingerprinter`认证。
