---
title: 'Linux Bash: 重复执行一个命令N次'
date: 2018-02-28 00:00:00
tags: ["Linux"]
abbrlink: repeat-run-command-use-linux-bash
img: ""
comments: false
---

### 方法一：

```
$ for n in {1..7}; do echo "Hello World!"; done
```



## 方法二：
在`~/.bashrc`文件中创建一个run函数：

```bash
function run() {
    number=$1
    shift
    for n in $(seq $number); do
      $@
    done
}
```
使生效：`# source ~/.bashrc`。使用方式：
```
# run 7 echo "Hello World!"

```
