---
title: C#关于async和await常问的问题
date: 2018-04-12 00:00:00
tags: ["C#"]
abbrlink: csharp-async-await-faqs
img: ""
comments: false
---

### 问题一：是不是写了async关键字的方法就代表该方法是异步方法，不会堵塞线程呢？

答: 不是的，对于只标识async关键字的(指在方法内没有出现await关键字)的方法,调用线程会把该方法当成同步方法一样执行,所以然而会堵塞GUI线程,只有当async和await关键字同时出现，该方法才被转换为异步方法处理。



### 问题二：“async”关键字会导致调用方法用线程池线程运行吗?

答: 不会,被async关键字标识的方法不会影响方法是同步还是异步运行并完成,而是，它使方法可被分割成多个片段，其中一些片段可能异步运行，这样这个方法可能异步完成。这些片段界限就出现在方法内部显示使用”await”关键字的位置处。所以，如果在标记了”async”的方法中没有显示使用”await”，那么该方法只有一个片段，并且将以同步方式运行并完成。在await关键字出现的前面部分代码和后面部分代码都是同步执行的(即在调用线程上执行的，也就是GUI线程，所以不存在跨线程访问控件的问题)，await关键处的代码片段是在线程池线程上执行。总结为——使用async和await关键字实现的异步方法，此时的异步方法被分成了多个代码片段去执行的，而不是像之前的异步编程模型(APM)和EAP那样，使用线程池线程去执行一整个方法。

关于更多async和await关键字的常问问题可以查看——[Async/Await FAQ](https://blogs.msdn.microsoft.com/pfxteam/2012/04/12/asyncawait-faq/ "Async/Await FAQ")和中文翻译——[（译）关于async与await的FAQ](http://www.cnblogs.com/heyuquan/archive/2012/11/30/2795859.html "（译）关于async与await的FAQ")
