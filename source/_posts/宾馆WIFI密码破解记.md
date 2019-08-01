---
title: 宾馆WIFI密码破解记
date: 2018-01-21 00:00:00
tags: ["WIFI"]
abbrlink: crack-hotel-wifi-password
img: ""
comments: false
---

今天在家，无线不能用了，正好家附近有个宾馆，电脑提示WIFI可以连接，看了一下，没有加密，所以直接连上去了，正要打开BAIDU查资料，没想到跳出一个网页。这下完啦，肯定是个WIFI管理系统一类的东东。有一个大大的按钮，点开后开一个弹出层，要求输入密码。心想这下完啦，这家伙骗MAC地址的。但是本着贼不走空的行业道德，随便输入几个常见密码和宾馆名字一类的东东都是提示出错，试了几次后放弃了，社工失败。于是打了FIREBUG看下信息是怎么发送的，在NETWORK查看请求后发现，密码没走网络验证。 这下好了，没走网络肯定就是本地验证，看代码就成了，密码肯定在代码里用JS验证的。于是查看按钮点击事件后发现这样一个事件。



```javascript
$("#wifiPwdCheck").unbind("click").bind("click",function(){
	if($.md5($("#wifi_passwd").val())!=wifi_pwd)
	{
		showinfo("WiFi密码不正确，请向工作人员询问密码~","error");
		return false;
	}else{
		$("#J_popbox_reg .ico_popclose").trigger("click");
		loginLogic();
	}
});
```
哈哈，直接控制台调用`loginLogic();`这个方法不就成了。 可是，妈蛋，控制台居然提示这个方法找不着。这下想代码段肯定找错啦。再看别的下手，从IF判断语句看出是把我输入的密码MD5后跟`wifi_pwd`这个变量进行对比，哦了，那就找wifi_pwd,发现这个代码。

```javascript 
var site_url = "http://portal.wangge.cc/api", gw_id = "D4EE07051FE4", location_name = "北京鑫顺居宾馆", ping_url = "http://www.baidu.com/img/baidu_jgylogo3.gif", is_debug = 'false', is_debug_mode = 'false', wifi_pwd = "08af3cb2ddfafec3c9e5afb4a1db0c3a", is_mikrotik = 'false', query_url_params = "_s=D2VSdwEIVDNSYwIxBXMKald0AnIAZVxuVjBXZlMuCzBbYAZsWyNVYgZqA21XfVdjA3UEYAkmUw8ALwNvU3IHeQ8%2FUjIBZ1RkUjcCcwVmCnhXWAJoADxcYlZNV2BTRQtEW2YGY1s9VWYGYgMSVxZXZgN1BGoJMFMzAGIDYVM4BzcPYFJiAW1UMVJhAm8FMQo7Vz0CZwBrXGVWP1dk&amp;_r=1395418515", mac_local = '', ip_local = '', is_tablet = 'false', location_id = '8126', is_show_365 = '1', dh_365_ad_id = '74', mac = 'a8bbcf04f360';
```

<p>原来直接是一个变量。。。技术人员真够操蛋的，这有难度么？？？？心想着直接复制到CMD5.ORG进行破解，好事来啦，此HASH可以找着对应的明文，操蛋的事儿也来啦，此条破解记录要收费，多少年没玩渗透了，CMD5怎么进化的这么资本主义啦。好吧，抽根烟后，理了下思路，突然发现，自己居然绕了这么大一个圈，直接给这个变量重新赋值不就完了，真想抽自己一巴掌。简单的生成一个MD5 HASH</p>
```php
#!/usr/bin/php
<?php
$code = "admin";
echo md5($code);
echo "\n";
```

拿到一个MD5 HASH后直接在console面板运行 wifi_pwd="xxx",输入密码后直接提示成功登入。好吧，登入后我又想起自己一件比较二的事儿来，我TM为毛要用PHP再去生成一个HASH去，他不是直接给了函数实现了么。。。。好吧，又绕了一下。
```
wifi_pwd=$.md5("admin")
```
这样儿运行就可以啦。。。 结语：这里按照文章的说法是要写点东西来总结的，鉴于这次破解自己太二了表现，就不总结啦。
