---
title: Git修改远程仓库地址
date: 2018-04-04 00:00:00
tags: ["Git"]
abbrlink: git-modify-remote-origin-url
img: ""
comments: false
---

## 方法有三种：

### 修改命令
```
git remote set-url origin [url]
```

### 先删后加
```
git remote rm origin
git remote add origin [url]
```

### 直接修改config文件
```
vi .git/config
```
