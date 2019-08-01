---
title: PHP中的强制类型转换
date: 2018-01-04 00:00:00
tags: ["php"]
abbrlink: php-force-variable-type-convert
img: ""
comments: false
---

PHP是一个弱类型的编程语言， 对一一些操作PHP会根据上下文的情况帮助我们完成对数据类型的转换， 但对于实际编程中一些情况需要用的强制的类型转换， 对于类型的强制转换PHP提供了两种方式:

1. 使用变量类型来对变量进行转换
2. 使用settype()函数来对PHP实现类型转换



先看下PHP中提供的几种变量类型

1. 四种标量：`integer` `float`（有时也做double）, `string` `boolean`
2. 两种复合类型： `array`, `object`
3. 两种特殊类型：`resource` `Null`
4. 还有一些伪类型：`mixed`, `callback`, `number`等。

如何去查看一个变量的类型呢？？ 看他的值么？ PHP里有现在的函数呀， 让PHP告诉你不就可以了：`gettype()` 函数可以快速获取一个值的类型， 但是不推荐在生产环境中这么使用， 下面再提供几种判定变量类型的函数：
 `is_float()`、`is_int()`、`is_integer()`、`is_string()` 和 `is_object()`, 这些函数从字面上就可以知道是做什么用的了吧， 返回的都是BOOL类型， 如果匹配类型返回true, 反之false.

上面的几个函数可以用于对变量类型的严格判断。

好了， 说了这么多，扯远了， 下面就看下怎么转换变量类型吧。
上面说过gettype()是不推荐使用的， 但是跟他相反的`settype()`就不会啦，

```php
//返回一个Boolean类型，成功返回 true, 失败返回 false
bool settype ( mixed $var , string $type )
```

怎么样是不是在其中看到一个伪变量类型“mixed”？它的意思就是可以接收各种类型的变量。后面的 $type 参数是重点， 他可能取以下值

- boolean （或为"bool"，从 PHP 4.2.0 起）
- integer （或为"int"，从 PHP 4.2.0 起）
- float （只在 PHP 4.2.0 之后可以使用，对于旧版本中使用的"double"现已停用）
- string
- array
- object
- null （从 PHP 4.2.0 起）


看一个小例子：
```php
<?php
$foo = "5bar"; // string
$bar = true;   // boolean

settype($foo, "integer"); // $foo 现在是 5   (integer)
settype($bar, "string");  // $bar 现在是 &quot;1&quot; (string)
```

`settype()`是一个比较简单的函数吧， 但是不够好看， 用起来也不爽，我个人喜欢用下面的方法来对一个变量类型完成转换。


- (int), (integer) - 转换为 整型(integer)
- (bool), (boolean) - 转换为 布尔型(boolean)
- (float), (double), (real) - 转换为 浮点型(float)
- (string) - 转换为 字符串(string)
- (binary) - 转换为二进制 字符串(string) (PHP 6)
- (array) - 转换为 数组(array) 
- (object) - 转换为 对象(object)
- (unset) - 转换为 NULL (PHP 5)


使用上面转换类型的好处是。。。。呃。。。少打字。。。我是个懒人。。

使用例子： `$a = (int)$a;`
这样就完成了对$a类型的转换。
当然这些是一些标准的类型转换， 后面还有操蛋的， 若知后事如何，等下回说吧。:))..
