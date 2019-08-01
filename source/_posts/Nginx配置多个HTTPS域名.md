---
title: Nginx配置多个HTTPS域名
date: 2018-02-06 00:00:00
tags: ["Nginx"]
abbrlink: muilti-nginx-https-domain
img: ""
comments: false
---

最近在玩微信小程序，手头有：

*   一台云服务器：CentOS 7
*   多个一级域名

开发测试过程中，因为某些原因，想要让手头的A、B域名同时指向云服务器的443端口，支持HTTPS。

Nginx支持TLS协议的SNI扩展（同一个IP上可以支持多个不同证书的域名），只需要重新安装Nginx，使其支持TLS即可。

### 安装Nginx

```source-shell
[root]#  wget http://nginx.org/download/nginx-1.12.0.tar.gz
[root]#  tar zxvf nginx-1.12.0.tar.gz
[root]#  cd nginx-1.12.0
[root]#  ./configure --prefix=/usr/local/nginx --with-http_ssl_module \
--with-openssl=./openssl-1.0.1e \
--with-openssl-opt="enable-tlsext"

```

备注：在安装的过程中发现，云服务器的环境中缺少一些库，下载后，重新执行Nginx的`./configure`指令，具体操作如下：



```source-shell
[root]#  wget https://nchc.dl.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.gz
[root]#  tar zxvf pcre-8.35
[root]#  yum -y install gcc
[root]#  yum -y install gcc-c++
[root]#  yum install -y zlib-devel

[root]#  ./configure --prefix=/usr/local/nginx --with-http_ssl_module \
--with-openssl=./openssl-1.0.1e \
--with-openssl-opt="enable-tlsext" \
--with-pcre=./pcre-8.35

```

### 配置Nginx

在购买域名的时候，如果域名提供商有免费的SSL证书，就直接用；如果没有的话，可以使用 **Let's Encript** 生成免费的CA证书。

打开Nginx的配置：`vi /etc/nginx/nginx.conf`

```source-shell

	...
	server {
		listen       443 ssl;
		listen       [::]:443 ssl;
		server_name  abc.com;
		root         /usr/share/nginx/html;

		ssl_certificate "/root/keys/abc.com.pem";
		ssl_certificate_key "/root/keys/abc.com.private.pem";
		include /etc/nginx/default.d/*.conf;

		location / {
		}
		error_page 404 /404.html;
		    location = /40x.html {
		}
		error_page 500 502 503 504 /50x.html;
		    location = /50x.html {
		}
	}

	server {
		listen       443 ssl;
		listen       [::]:443 ssl;
		server_name  def.com;
		root         /usr/share/nginx/html;

		ssl_certificate "/root/keys/def.com.pem";
		ssl_certificate_key "/root/keys/def.com.private.pem";
		include /etc/nginx/default.d/*.conf;

		location / {
		}
		error_page 404 /404.html;
		    location = /40x.html {
		}
		error_page 500 502 503 504 /50x.html;
		    location = /50x.html {
		}
	}

```

配置完成后，重新加载Ngixn：`nginx -s reload`

### 申请免费的CA证书

对于没有SSL证书的情况，可以用下面的方法免费获得CA证书——Let's Encript。

**步骤1: 安装 Let's Encrypt 官方客户端——CetBot**

```
[root]#  yum install -y epel-releasesudo 
[root]#  yum install -y certbot

```

**步骤2: 配置Nginx的配置文件，在 Server 模块（监听80端口的）添加下面配置：**

CertBot在验证服务器域名的时候，会生成一个随机文件，然后CertBot的服务器会通过HTTP访问你的这个文件，因此要确保你的Nginx配置好，以便可以访问到这个文件。

```
server {
  	listen       80 default_server;

  	...

	location ^~ /.well-known/acme-challenge/ {   
		default_type "text/plain";   
		root     /usr/share/nginx/html;
	}

	location = /.well-known/acme-challenge/ {   
		return 404;
	}
}

```

重新加载Nginx： `nginx -s reload`

**步骤3: 申请SSL证书**

```
[root]# certbot certonly --webroot -w /usr/share/nginx/html/ -d your.domain.com

```

安装过程中，会提示输入邮箱，用于更新CA证书的。

安装成功后，默认会在 `/etc/letsencrypt/live/your.domain.com/` 会生成CA证书。

```
|-- fullchain.pem 
|-- privkey.pem

```

**步骤4: 配置Nginx**

```
server {
	listen       443 ssl;
	listen       [::]:443 ssl;
	server_name  def.com;
	root         /usr/share/nginx/html;

	ssl_certificate "/etc/letsencrypt/live/your.domain.com/fullchain.pem";
	ssl_certificate_key "/etc/letsencrypt/live/your.domain.com/privkey.pem";
	include /etc/nginx/default.d/*.conf;

	location / {
	}
	error_page 404 /404.html;
	    location = /40x.html {
	}
	error_page 500 502 503 504 /50x.html;
	    location = /50x.html {
	}
}

```

配置完，重新加载Nginx

步骤5: 自动更新证书

在命令行先进行模拟更新证书

```
certbot renew --dry-run

```

如果模拟更新成功，则 使用 `crontab -e` 命令来启用自动更新任务：

```
[root]# crontab -e

30 2 * * 1 /usr/bin/certbot renew  >> /var/log/le-renew.log

```

### 相关参考

*   [Let's Encript](https://letsencrypt.org)
*   [Nginx下载连接](http://nginx.org/en/download.html)
*   [crontab](http://baike.baidu.com/link?url=cwI6PVyqw5tCtCT7teLpI_MiJRqtSG_oJwVMgxpX5DmzT6Vs0CaqJIVwKcXWcDgtVE8saFylcURpgBI3qcfw7K)
