---
title: Let't Encrypt Nginx SSL反向代理配置
date: 2018-03-03 00:00:00
tags: ["Nginx"]
abbrlink: lets-encrytp-ssl-reverse-proxy-configurations
img: ""
comments: false
---

Let's Encrypt 证书更新`crontab -e`

```
30 2 * * 1 /usr/bin/certbot renew  >> /var/log/le-renew.log
```



NGINX服务器配置：
```
upstream www_711xd_com {
       server 127.0.0.1:8096;
}


server {
        listen 80; # must listen
        listen 443 ssl;
        server_name 711xd.com www.711xd.com;

        access_log /var/log/nginx/711xd.access.log;
        error_log /var/log/nginx/711xd.error.log;

		# for certbot autorenew
        location ^~ /.well-known/acme-challenge/ {
           default_type "text/plain";
           root     /usr/share/nginx/html;
        }


		# redirect www pefix domain
        if ($host = 711xd.com) {
                return 301 https://www.$host$request_uri;
        }

        if ($scheme = http) {
                return 301 https://$host$request_uri;
        }


        ssl on;
        ssl_certificate /etc/letsencrypt/live/www.711xd.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/www.711xd.com/privkey.pem;
        ssl_trusted_certificate /etc/letsencrypt/live/www.711xd.com/chain.pem;


        include "conf/antispider.conf";

        location / {
                proxy_pass http://www_711xd_com;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header User-Agent $http_user_agent;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
}
```
