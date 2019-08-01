---
title: 聊聊CSS3 Flexbox
date: 2018-02-05 00:00:00
tags: ["Flexbox"]
abbrlink: point-for-css3-flexbox
img: ""
comments: false
---

本文涉及内容如下： flexbox的基本概念、容器属性学习、项目属性学习、实战演练。 flexbox 堪称布局神器，但属性实在太多让人无从下手，因此将自己所学的知识做个总结。

## 基本概念

flexbox是Flexible Box的缩写，译为弹性布局。flex 布局主要是让容器中的子项目可以根据配置改变自身的宽高及顺序，以最佳的方式填充容器，达到`弹性`的目的。容器有横轴和纵轴来划分容器，横轴默认为水平方向纵轴为垂直方向。

![聊聊CSS3 Flexbox](http://oss.ipaoyun.com/blog/1-1512030664.png)



## 容器属性

容器属性用来控制布局的大方向。

*   flex-direction
*   flex-wrap
*   flex-flow
*   justify-content
*   align-items
*   align-content

### flex-direction

flex-direction属性决定主轴方向（即项目的排列方向）。 row | row-reverse | column | column-reverse
[code demo](https://github.com/xiyuanyuan/flex-demo/blob/master/flex-direction.html) [preview](http://xixitoday.com/flex-demo/flex-direction.html)

### flex-wrap

该属性用来控制，当容器的主轴方向放不下所有项目时该如何处理。wrap | wrap-reverse | no-wrap, no-wrap 为默认值。
[code demo](https://github.com/xiyuanyuan/flex-demo/blob/master/flex-wrap.html) [preview](http://xixitoday.com/flex-demo/flex-wrap.html)

### flex-flow

flex-flow 是 flex-direction 和 flex-wrap 两个属性的简写，你要是记不住也不必强求。默认值为row nowrap。

### justify-content

justify-content定义子项目在主轴上的对齐方式。可以联想下 `text-align`。flex-start | flex-end | center | space-between | space-around
需要注意的是：space-around的两边的边距要比中间的边距要小一些。
[code demo](https://github.com/xiyuanyuan/flex-demo/blob/master/justify-content.html) [preview](http://xixitoday.com/flex-demo/justify-content.html)

### align-items

justify-content定义子项目在纵轴上的对齐方式。 flex-start | flex-end | center | baseline | stretch
[code demo](https://github.com/xiyuanyuan/flex-demo/blob/master/align-items.html) [preview](http://xixitoday.com/flex-demo/align-items.html)

### align-content

> align-content属性定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用。

也就是说只有当 wrap生效时，该属性才有存在的意义。flex-start | flex-end | center | space-between | space-around | stretch
[code demo](https://github.com/xiyuanyuan/flex-demo/blob/master/align-content.html) [preview](http://xixitoday.com/flex-demo/align-content.html)

以上就是flex 布局所涉及的所以容器属性。下一小结，我们将共同学习项目相关属性。

## 项目属性

项目属性用来控制容器中的项目自身的位置和伸缩。

*   order
*   flex-grow
*   flex-shrink
*   flex-basis
*   flex
*   align-self

### order

order 用来控制 flex 项目自身的摆放顺序，默认值为0，可以为负数，值越小项目越靠前摆放。
[code demo](https://github.com/xiyuanyuan/flex-demo/blob/master/order.html) [preview](http://xixitoday.com/flex-demo/order.html)

### flex-grow

flex-grow控制项目的放大比例，默认为0，不放大。值得注意的是放大的比例是相对于剩余的空间而言，而不是项目自己的大小。
[code demo](https://github.com/xiyuanyuan/flex-demo/blob/master/flex-grow.html) [preview](http://xixitoday.com/flex-demo/flex-grow.html)

### flex-shrink

flex-shrink 与 flex-grow 类似，主要用来控制项目的缩小比例，默认值为1，同比缩小。
[code demo](https://github.com/xiyuanyuan/flex-demo/blob/master/flex-shrink.html) [preview](http://xixitoday.com/flex-demo/flex-shrink.html)
flex-grow 和 flex-shrink 都是按照剩余空间进行放大缩小的，而不是自身。大家记住瘦死的骆驼比马大。

### flex-basis

flex-basis 很好理解，若横轴是主轴，flex-basis 可以当做 width 来使用；若纵轴是主轴，flex-basis 可以当做 height 来使用。个人感觉 flex-basis width 和 height 更灵活。

### flex

flex 属性是 flex-grow flex-shrink flex-basis 三个属性的缩写。同样的原则，为了不增加大家的学习难度，不会不必强求。今天只向大家解释一下 flex: 1;当 flex的值为整数是它代表 flex-grow: 数值； flex-shrink采用默认值1；flex-basis:为0%。

```
.item {flex: 1;}
.item {
    flex-grow: 1;
    flex-shrink: 1;
    flex-basis: 0%;
}
```

那么大家思考一下flex: 2;等同于什么？

```
.item {flex: 2;}
.item {
    flex-grow: 2;
    flex-shrink: 1;
    flex-basis: 0%;
}
```

### align-self

align-self控制自身在侧重的对齐方式，和容器属性 align-items 类似，当然了，优先机肯定是高于他的爸爸的。auto | flex-start | flex-end | center | baseline | stretch
[code demo](https://github.com/xiyuanyuan/flex-demo/blob/master/align-self.html) [preview](http://xixitoday.com/flex-demo/align-self.html)

以上项目的属性和练习也完成了，接下来使用 flex 布局实现我们日常工作中常见的三个需求。

## 实战

实现等宽布局，即使项目被删除和添加也不需要更改 css 的代码，常用来实现导航[code demo](https://github.com/xiyuanyuan/flex-demo/blob/master/Demo-03.html) [preview](http://xixitoday.com/flex-demo/Demo-03.html)
垂直水平居中，该需求是特别常见的使用 flex 特别容易。[code demo](https://github.com/xiyuanyuan/flex-demo/blob/master/Demo-02.html) [preview](http://xixitoday.com/flex-demo/Demo-02.html)
等高布局，当左右两个框的高度不定时，我们可以考虑使用 flex 实现。[code demo](https://github.com/xiyuanyuan/flex-demo/blob/master/Demo-01.html) [preview](http://xixitoday.com/flex-demo/Demo-01.html)

[FLEXBOX FROGGY游戏检验一下自己对 flexbox 的理解](http://flexboxfroggy.com/#zh-cn)
