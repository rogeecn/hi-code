---
title: Yii2.0中接收Payload参数值
date: 2018-02-02 00:00:00
tags: ["php"]
abbrlink: receive-payload-params-from-requery
img: ""
comments: false
---

在一些前端Ajax请求中，发送到后端数据以Playload，这时在后端直接用`Yii::app()->getRequest()->get("xxx")`是无法直接获取到值的。需要在配置文件中加入相应的`parser`才可以实现直接取参数，修改配置文件`main.php`配置如下：

```php
'components'=>[
	//...
	'request'      => [
		'parsers' => [
			'application/json' => 'yii\web\JsonParser',
			'text/json'        => 'yii\web\JsonParser',
		],
	],
]
```

这时通过`Yii::app()->getRequest()->get("xxx")`方法就可以获取前端传递参数值了。
