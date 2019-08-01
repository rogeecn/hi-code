---
title: Dockerfile中CMD和ENTRYPOINT的区别
date: 2018-02-26 00:00:00
tags: ["Docker"]
abbrlink: dockerfile-difference-between-cmd-entrypoint
img: ""
comments: false
---

Dockerfile里有 CMD 与 ENTRYPOINT 两个功能咋看起来很相似的指令，开始的时候觉得两个互用没什么所谓，但其实并非如此：


## CMD指令：
> The main purpose of a CMD is to provide defaults for an executing container.

CMD在容器运行的时候提供一些命令及参数，用法如下：
```
CMD ["executable","param1","param2"] (exec form, this is the preferred form)
CMD ["param1","param2"] (as default parameters to ENTRYPOINT)
CMD command param1 param2 (shell form)
```
第一种用法：运行一个可执行的文件并提供参数。
第二种用法：为ENTRYPOINT指定参数。
第三种用法(shell form)：是以”/bin/sh -c”的方法执行的命令。



如你指定:
```
CMD [“/bin/echo”, “this is a echo test ”]
```
build后运行(假设镜像名为ec):
```
docker run ec
```
就会输出: `this is a echo test`

是不是感觉很像开机启动项，你可以暂时这样理解。


注意点：

**docker run命令如果指定了参数会把CMD里的参数覆盖**： （这里说明一下，如：`docker run -it ubuntu /bin/bash` 命令的参数是指`/bin/bash` 而非 `-it` ,`-it`只是`docker` 的参数，而不是容器的参数，以下所说参数均如此。）

同样是上面的ec镜像启动：
```
docker run ec /bin/bash
```
就不会输出：`this is a echo test`，因为CMD命令被`/bin/bash`覆盖了。

 
## ENTRYPOINT 　

字面意思是进入点，而它的功能也恰如其意。

> An ENTRYPOINT allows you to configure a container that will run as an executable.它可以让你的容器功能表现得像一个可执行程序一样。

容器功能表现得像一个可执行程序一样，这是什么意思呢？

直接给个例子好说话：

### 例子一：

使用下面的ENTRYPOINT构造镜像：
```
ENTRYPOINT ["/bin/echo"]
```
那么docker build出来的镜像以后的容器功能就像一个`/bin/echo`程序：

比如我build出来的镜像名称叫imageecho，那么我可以这样用它：
```
docker  run  -it  imageecho  “this is a test”
```
这里就会输出`this is a test`这串字符，而这个imageecho镜像对应的容器表现出来的功能就像一个echo程序一样。 你添加的参数`this is a test`会添加到ENTRYPOINT后面，就成了这样　`/bin/echo "this is a test"` 。现在你应该明白进入点的意思了吧。

 

### 例子二：
```
ENTRYPOINT ["/bin/cat"]
```
构造出来的镜像你可以这样运行(假设名为st)：
```
docker run -it st /etc/fstab
```
这样相当： `/bin/cat  /etc/fstab` 这个命令的作用。运行之后就输出`/etc/fstab`里的内容。

 

ENTRYPOINT有两种写法：    

写法一：`ENTRYPOINT ["executable", "param1", "param2"]` (the preferred exec form)
写法二：`ENTRYPOINT command param1 param2` (shell form)

你也可以在docker run 命令时使用–entrypoint指定（但是只能用写法一）。
