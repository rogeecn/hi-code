---
title: Sass编译出现Invalid US-ASCII character
date: 2018-02-27 00:00:00
tags: ["SASS"]
abbrlink: sass-invalid-us-ascii-charater
img: ""
comments: false
---

编译scss文件时，如果出现如下错误
```
Error: Invalid US-ASCII character "\xC2"  
        on line 63 of src/assets/_scss/partials/_post-list.scss
        from line 36 of src/assets/_scss/style.scss
  Use --trace for backtrace.
```
## 办法一
在报错的scss文件顶部加上编码类型即可：
```
@charset "UTF-8";
```

### 办法二
加运行参数
```
sass --watch style.scss:style.min.css --style compressed -E "UTF-8"
```
