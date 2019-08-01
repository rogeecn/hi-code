---
title: Docker Alpine 镜像无法编译C语言模块问题解决
date: 2018-05-15 00:00:00
tags: ["Docker"]
abbrlink: docker-alpine-jing-xiang-wu-fa-bian-yi-c-yu-yan-mo-kuai-wen-ti-jie-jue
img: ""
comments: false
---

Docker Alpine 镜像中手动编译PHP时碰到如下问题

```bash
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking for a sed that does not truncate output... /bin/sed
checking for cc... cc
checking whether the C compiler works... no
configure: error: in `/root/qh360_license':
configure: error: C compiler cannot create executables
See `config.log' for more details
```


执行:` apk add gcc g++ make libffi-dev openssl-dev`
安装相应模块，即可解决。
