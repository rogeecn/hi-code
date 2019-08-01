---
title: 使用Nginx反向代理实现微信公众号图片防盗链的破解
date: 2018-01-29 00:00:00
tags: ["Nginx"]
abbrlink: use-nginx-reverse-proxy-to-access-wechat-mp-images
img: ""
comments: false
---

微信公众号图片现在对非TX系的域名下直接引用加入了防盗链机制，我们可以通过使用Nginx反向代理来进行访问，不过这么实现会有下面的问题：
- 流量走自己的服务器（建议网页配置LazyLoad，防止页面图片对服务器一次次加载造成浪费）
- 需要有自己的域名（直接使用服务器IP也可以）
- 国内要有主机（国外主机的速度让你有想死的冲动）



下面是我的配置代码：
```
server {
    listen 80;
    server_name img.domain.com; # 这里写你的图片域名

    access_log off;

    location / {
        resolver 114.114.114.114;

        set $proxy_to http:/$uri;
        proxy_set_header Referer http://mp.weixin.qq.com;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass $proxy_to;

        expires 365d;
    }
}
```

配置完`nginx -t`测试下没问题后`nginx -s reload`加载配置。
使用下面的链接格式测试微信的图片是否可以正常访问：
http://img.domain.com/m.qpic.cn/xxxxxx-xxxxx-xxxxx/

配置完成后，需要修改你的程序在文章内容输出时对图片链接进行替换：
```
$replaceContent = strtr($rawContent, [
    "http://m.qpic.cn"       => "http://img.domain.com/m.qpic.cn",
    "https://m.qpic.cn"      => "http://img.domain.com/m.qpic.cn",
    "http://a1.qpic.cn"      => "http://img.domain.com/a1.qpic.cn",
    "https://a1.qpic.cn"     => "http://img.domain.com/a1.qpic.cn",
    "http://mmbiz.qpic.cn"   => "http://img.domain.com/mmbiz.qpic.cn",
    "https://mmbiz.qpic.cn"  => "http://img.domain.com/mmbiz.qpic.cn",
    "http://mmbiz.qlogo.cn"  => "http://img.domain.com/mmbiz.qlogo.cn",
    "https://mmbiz.qlogo.cn" => "http://img.domain.com/mmbiz.qlogo.cn",
    "http://mmsns.qpic.cn"   => "http://img.domain.com/mmsns.qpic.cn",
    "https://mmsns.qpic.cn"  => "http://img.domain.com/mmsns.qpic.cn",
])
```
