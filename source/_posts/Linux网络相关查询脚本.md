---
title: Linux网络相关查询脚本
date: 2018-01-27 00:00:00
tags: ["Linux"]
abbrlink: linux-query-network-status-scripts
img: ""
comments: false
---

* 查看TCP连接状态
```
netstat -nat |awk '{print $6}'|sort|uniq -c|sort -rn
netstat -n | awk '/^tcp/ {++S[$NF]};END {for(a in S) print a, S[a]}' 或
netstat -n | awk '/^tcp/ {++state[$NF]}; END {for(key in state) print key,"\t",state[key]}'
netstat -n | awk '/^tcp/ {++arr[$NF]};END {for(k in arr) print k,"\t",arr[k]}'
netstat -n |awk '/^tcp/ {print $NF}'|sort|uniq -c|sort -rn
netstat -ant | awk '{print $NF}' | grep -v '[a-z]' | sort | uniq -c
```


(以上每一行实现的效果基本相同，在此列出不同的写法，方便对脚本写法的更深理解）




* 查找请求数请20个IP（常用于查找攻来源）
```
netstat -anlp|grep 80|grep tcp|awk '{print $5}'|awk -F: '{print $1}'|sort|uniq -c|sort -nr|head -n20
netstat -ant |awk '/:80/{split($5,ip,":");++A[ip[1]]}END{for(i in A) print A[i],i}' |sort -rn|head -n20
```


* 用tcpdump嗅探80端口的访问看看谁最高
```
tcpdump -i eth0 -tnn dst port 80 -c 1000 | awk -F"." '{print $1"."$2"."$3"."$4}' | sort | uniq -c | sort -nr |head -20
```


* 查找较多time_wait连接
```
netstat -n|grep TIME_WAIT|awk '{print $5}'|sort|uniq -c|sort -rn|head -n20
```


* 找查较多的SYN连接
```
netstat -an | grep SYN | awk '{print $5}' | awk -F: '{print $1}' | sort | uniq -c | sort -nr | more
```


* 根据端口列进程
```
netstat -ntlp | grep 80 | awk '{print $7}' | cut -d/ -f1
```


* 查看有多少个PHP-CGI进程活动
```
netstat -anp | grep php-cgi | grep ^tcp | wc -l
```


* 查看PHP-CGI占用内存的总数
```
total=0; for i in `ps -C php-cgi -o rss=`; do total=$(($total+$i)); done; echo "PHP-CGI Memory usage: $total kb"
```
