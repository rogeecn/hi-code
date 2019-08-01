---
title: Nginx禁止POST只允许GET请求的方法
date: 2018-05-11 00:00:00
tags: ["Nginx"]
abbrlink: nginx-jin-zhi-post-zhi-yun-xu-get-qing-qiu-de-fang-fa
img: ""
comments: false
---

Nginx 服务器可以使用下面的配置实现只请接受某一种请求，其它请求返回指定的状态码。

```
server {
        listen      80;
        server_name server_domain.com
        ...
		
        if ($request_method !~* GET|POST) {
            return 403;
        }
		
		...
}

```


下面这个代码就是屏蔽非GET、POST类型请求，返回XXX状态码。

```
 if ($request_method !~* GET|POST) {
	return 403;
}
```

或者
```
if ($request_method !~ ^(GET|POST)$ )
{
	return 403;
}
```

关于 `!~`等判断关键字的含义可以参考：[Nginx配置location 匹配规则](https://www.711xd.com/p/nginx-conf-location-rules "Nginx配置location 匹配规则")
