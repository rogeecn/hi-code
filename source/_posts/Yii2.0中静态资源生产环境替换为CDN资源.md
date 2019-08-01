---
title: Yii2.0中静态资源生产环境替换为CDN资源
date: 2018-02-01 00:00:00
tags: ["Yii2"]
abbrlink: replace-yii2-assets-files-to-cdn-resources
img: ""
comments: false
---

Yii2.0 通过使用Assets的方式注册静态资源来管理资源依赖。有些比较大的文件（如：jQueryUI,bootstrap）在生产环境中替换为CDN资源可以加快页面访问速度。

配置方式如下：
修改 `environment/product/xxxx/config/main-local.php`文件，在`common`配置中添加如下规则,

```php
'components'=>[
		// ...
        'assetManager' => [
            'bundles' => [
                'yii\web\JqueryAsset' => [
                    'jsOptions' => [
                        'position' => \yii\web\View::POS_HEAD,
                    ],
                    'js'        => [
                        '//cdn.staticfile.org/jquery/3.2.1/jquery.min.js',
                    ],
                ],
            ],
        ],
		// ...
]
```



对于其它资源，自行替换`yii\web\JqueryAsset`为相应包的`namespace`即可。

上面配置中，同样可以对资源加载位置进行配置，如：
```php
'jsOptions' => [
	'position' => \yii\web\View::POS_HEAD,
],
```
这个配置说明了，jQuery在页面的head中加载，同样的你可以配置为下面的值:

- `POS_HEAD` 在head标签内加载, 适用（文件，js）
- `POS_BEGIN` 在body开始加载， 适用（文件，js）
- `POS_END` 在body末尾加载， 适用（文件，js）
- `POS_READY` 在jQuery插件Ready事件中加载， 适用（js）
- `POS_LOAD` 在页面`onload`事件时加载， 适用（js）
