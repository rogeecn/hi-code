---
title: Nginx配置实现CORS跨域请求
date: 2018-02-22 00:00:00
tags: ["Nginx"]
abbrlink: nginx-config-to-access-cross-region-request
img: ""
comments: false
---

在开发的时候，我们经常会遇到跨域问题，我们通常的解决方案是使用CORS或是JSONP来解决，但是更常用的解决方案便是CORS了，因为JSONP只支持GET请求，关于CORS的相关知识可以参考[阮一峰的跨域资源共享 CORS](http://www.ruanyifeng.com/blog/2016/04/cors.html "阮一峰的跨域资源共享 CORS") 详解这篇文章。

如果浏览器控制台出现跨域相关的报错时，一般是后端没有允许跨域，所以需要后端服务器在代码层面设置下。当然有的时候后端并不想对代码进行改动，这个时候我们也可以直接在代理层nginx这里进行一些相关配置
比如我们要允许/api/v1下的相关接口支持跨域，我们可以这样编写nginx配置



```nginx
location ^~ /api/v1 {
	add_header 'Access-Control-Allow-Origin' "$http_origin"; 
	add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS'; 
	add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type    '; 
	add_header 'Access-Control-Allow-Credentials' 'true'; 
	if ($request_method = 'OPTIONS') { 
		add_header 'Access-Control-Allow-Origin' "$http_origin"; 
		add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS'; 
		add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type    '; 
		add_header 'Access-Control-Allow-Credentials' 'true'; 
		add_header 'Access-Control-Max-Age' 1728000; # 20 天 
		add_header 'Content-Type' 'text/html charset=UTF-8'; 
		add_header 'Content-Length' 0; 
		return 200; 
	} 
    # 这下面是要被代理的后端服务器，它们就不需要修改代码来支持跨域了
	proxy_pass http://127.0.0.1:8085; 
	proxy_set_header Host $host; 
	proxy_redirect off; 
	proxy_set_header X-Real-IP $remote_addr; 
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
	proxy_connect_timeout 60; 
	proxy_read_timeout 60; 
	proxy_send_timeout 60; 

}
```
