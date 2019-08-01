---
title: 微信开发使用Nginx+frp反向代理实现公网访问内网测试网页
date: 2018-05-24 00:00:00
tags: ["frp"]
abbrlink: shi-yong-frp-fan-xiang-dai-li-shi-xian-gong-wang-fang-wen-nei-wang-ce-shi-wang-ye
img: ""
comments: false
---

`frp` 是一款内网穿透工具， 相关介绍可以查看`github`主页，这里不再一一介绍 ，主页地址：https://github.com/fatedier/frp

目前我需要开发联调微信公众号开发，于是想办法把内网的网页发布到公网的一个域名上实现实时真实数据调试。

## 配置

`frp`有两套配置文件，一个是`server`端， 一个是 `client` 端。

### server 端口配置文件

```ini
[common]
bind_addr = 0.0.0.0 // 这里我需要公网所有数据访问，所以是绑定这个
bind_port = 7777 //公网访问的端口


# if you want to support virtual host, you must set the http port for listening (optional)
# Note: http port and https port can be same with bind_port
vhost_http_port = 7777 // 如果是需要转发http请求的话，这个字段必填，否则client会报错

# if subdomain_host is not empty, you can set subdomain when type is http or https in frpc's configure file
# when subdomain is test, the host used by routing is test.frps.com
subdomain_host = 711xd.com // 发布公网的根域名
```

配置文件设置完成后在服务器运行 `./frps -c frps.711xd.com.ini`, 完成启动，看下有没有什么报错。

### client 端配置文件

```ini
[common]
server_addr = xx.xx.xx.xx  // 上面运行 frps 的服务器公网地址
server_port = 7777 // 上面公网绑定的端口

[web]
type = http //类型
local_ip = 127.0.0.1 // 转发的地址，如果在内网其它机器做的转发，需要把地址换成自己的内网iP
local_port = 80 // 本机 WEB服务监听端口
use_encryption = false // 是否加密
use_compression = true //是否压缩

subdomain = mp // 使用哪个子域名
host_header_rewrite = hdzy.local // 本地开发使用的host
```

配置完成后 运行`./frpc -c frpc.hdzy.local.ini`, 这时候访问 `http://mp.711xd.com:7777`就可看到本地 `http://hdzy.local`的网页服务了。

## 如何去除端口号访问？

现在可以使用域名+端口的方式访问网页了，但是微信配置输入框中提示是不支持再端口的网页链接的，我们还需要使用 nginx 的反向代理来把端口与隐藏起来。

```nginx
server {
        listen 80;
        server_name mp.711xd.com;

        location / {
                proxy_pass http://127.0.0.1:7777$request_uri;
                proxy_set_header Host $host;
                proxy_set_header X-Forwarded-For $remote_addr;
        }
}
```

使用 `Nginx` 的反向代理功能把端口号隐藏起来。
现在访问下不带端口的域名试试
你会回来打赏的！
