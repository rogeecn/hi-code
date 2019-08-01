---
title: Git 保存用户名和密码
date: 2018-05-20 00:00:00
tags: ["Git"]
abbrlink: git-bao-cun-yong-hu-ming-he-mi-ma
img: ""
comments: false
---

Git可以将用户名，密码和仓库链接保存在硬盘中，而不用在每次push的时候都输入密码。
保存密码到硬盘一条命令就可以

## 操作步骤
```bash
$ git config credential.helper store
```



当 `git push` 的时候输入一次用户名和密码就会被记录
参考
```bash
$ man git | grep -C 5 password
$ man git-credential-store
```

这样保存的密码是明文的，保存在用户目录的 `.git-credentials` 文件中
```bash
$ file ~/.git-credentials
$ cat ~/.git-credentials
```

你也可以用 `git config credential.helper 'cache --timeout=36000'`，36000秒内不关机不用输入git密码

## 引用

EXAMPLES
The point of this helper is to reduce the number of times you must type your username or password. For example:
```bash
$ git config credential.helper store
$ git push http://example.com/repo.git
Username: <type your username>
Password: <type your password>



[several days later]
$ git push http://example.com/repo.git
[your credentials are used automatically]
```



The .git-credentials file is stored in plaintext. Each credential is stored on its own line as a URL like:

```
https://user:pass@example.com
```
