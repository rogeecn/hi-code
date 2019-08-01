---
title: PHP即时输出缓冲区内容实现实时输出
date: 2018-01-07 00:00:00
tags: ["php"]
abbrlink: flush-php-cache-to-real-output-content-to-client
img: ""
comments: false
---

```php
ob_start(); //打开输出缓冲区 
ob_end_flush(); 
ob_implicit_flush(1); //立即输出
for($i=0;$i<100;$i++){
	echo str_repeat(" ",4096); //确保足够的字符，立即输出，Linux服务器可以去掉这个语句
	echo $i."<br>";
	sleep(1);
}
```

这段代码如果你运行的话是不会输出到的100的， 因为PHP有30S的最大执行限制， 如果要输出到100的话可以设置`set_time_limit`来改变PHP脚本运行的时间限制。
