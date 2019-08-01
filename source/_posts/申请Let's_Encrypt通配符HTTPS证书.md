---
title: 申请Let's Encrypt通配符HTTPS证书
date: 2018-04-06 00:00:00
tags: ["Nginx"]
abbrlink: apply-let-s-encrypt-common-https-cert
img: ""
comments: false
---

Let's Encrypt 发布的 ACME v2 现已正式支持通配符证书，接下来将为大家介绍怎样申请，Let's go.

注 本教程是在centos 7下操作的，其他Linux系统大同小异。

### 1.获取`certbot-auto`
```
# 下载
wget https://dl.eff.org/certbot-auto

# 设为可执行权限
chmod a+x certbot-auto
```



### 2.开始申请证书
> 注xxx.com请根据自己的域名自行更改

```
./certbot-auto --server https://acme-v02.api.letsencrypt.org/directory -d "*.xxx.com" --manual --preferred-challenges dns-01 certonly
```

执行完这一步之后，会下载一些需要的依赖，稍等片刻之后，会提示输入邮箱，随便输入都行【该邮箱用于安全提醒以及续期提醒】

![](http://oss.ipaoyun.com/blog/1-1521082430.png)

注意，申请通配符证书是要经过DNS认证的，按照提示，前往域名后台添加对应的DNS TXT记录。添加之后，不要心急着按回车，先执行dig xxxx.xxx.com txt确认解析记录是否生效，生效之后再回去按回车确认
![](http://oss.ipaoyun.com/blog/2-1521082441.png)
到了这一步后，大功告成！！！ 证书存放在/etc/letsencrypt/live/xxx.com/里面

要续期的话，执行`certbot-auto renew`就可以了
![](http://oss.ipaoyun.com/blog/3-1521082454.png)
注：经评论区 ddatsh 的指点，这样的证书无法应用到主域名xxx.com上，如需把主域名也增加到证书的覆盖范围，请在开始申请证书步骤的那个指令把主域名也加上，如下： 需要注意的是，这样的话需要修改两次解析记录
```
./certbot-auto --server https://acme-v02.api.letsencrypt.org/directory -d "*.xxx.com" -d "xxx.com" --manual --preferr
```
![](http://oss.ipaoyun.com/blog/4-1521082469.png)
下面是一个nginx应用该证书的一个例子
```
server {
    server_name xxx.com;
    listen 443 http2 ssl;
    ssl on;
    ssl_certificate /etc/cert/xxx.cn/fullchain.pem;
    ssl_certificate_key /etc/cert/xxx.cn/privkey.pem;
    ssl_trusted_certificate  /etc/cert/xxx.cn/chain.pem;

    location / {
      proxy_pass http://127.0.0.1:6666;
    }
}
```
