---
title: MySQL服务器最大连接数的合理设置
date: 2018-02-18 00:00:00
tags: ["MySQL"]
abbrlink: the-rational-setting-of-the-maximum-number-of-connections-in-mysql-serve
img: ""
comments: false
---

MySQL服务器的连接数如何设置是我们都需要重点考虑的问题，合理的设置可以让服务器运行更加的稳定，下面就为您分析MySQL服务器最大连接数的合理设置，希望对您有所帮助。
MySQL服务器的连接数并不是要达到最大的100%为好，还是要具体问题具体分析，下面就对MySQL服务器最大连接数的合理设置进行了详尽的分析，供您参考。

我们经常会遇见“MySQL: ERROR 1040: Too many connections”的情况，一种是访问量确实很高，MySQL服务器抗不住，这个时候就要考虑增加从服务器分散读压力，另外一种情况是MySQL配置文件中max_connections值过小：



```bash
mysql> show variables like 'max_connections';
+-----------------+-------+
| Variable_name | Value |
+-----------------+-------+
| max_connections | 256 |
+-----------------+-------+
```

这台MySQL服务器最大连接数是256，然后查询一下服务器响应的最大连接数：

```bash
mysql> show global status like 'Max_used_connections';
```

MySQL服务器过去的最大连接数是245，没有达到服务器连接数上限256，应该没有出现1040错误，比较理想的设置是：

```bash
Max_used_connections / max_connections * 100% ≈ 85%
```

最大连接数占上限连接数的85%左右，如果发现比例在10%以下，MySQL服务器连接上线就设置得过高了。
