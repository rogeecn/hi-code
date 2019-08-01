---
title: 理解CSS3 Flexbox 布局
date: 2018-02-04 00:00:00
tags: ["Flexbox"]
abbrlink: css3-flexbox-layout-course
img: ""
comments: false
---

## Flexbox 让人困惑

有很多谈及 Flexbox 的文章，但依然有不少前端对此感到困惑。一方面，flex 相关的 CSS 属性繁多，影响到的具体效果也包含多个方面；另一方面，CSS 可以使用 **Shorthand properties** 风格的写法（例如最常见的 `background: url(images/bg.gif) no-repeat left top;`），很容易让新手弄不清具体含义。


这篇文章要讲的 Flexbox 当然还是 CSS3 规范中的弹性盒模型，不过写出前面一段，是因为我希望这篇文章可以解决那些问题——简单说，就是 Flexbox 让人困惑这样的问题。解决的方法，就是**理解它**。




## 什么时候，我们会想到弹性

为了理解弹性盒模型，先要从古龙的笔法开始。古龙是台湾的武侠小说大师，成就仅次于金庸。古龙描写的一些人物，非常深入人心，其犀利以及令人难忘的程度，甚至超过金庸。比如，有一位中年男性，在整部小说里，他的手非常重要，古龙对此有多次描写——“他的手指修长而有力”。而在不止一本书中，不止一两个情节中，有几位青年女性，她们的腿非常重要，古龙的描述是——“大腿结实而富有弹性”，或者“修长结实而富有弹性”。


轻呼一口气，思考一下，为什么描写手指的时候要说“修长而有力”，而描写大腿却是“结实而富有弹性”？这真是一个非常值得思考的问题。实际上，人的腿也可以是“修长而有力”的；而且，人的手指也是有弹性的。但古大师的描写绝非随意为之，其中的道理，可以随便找些**包含大腿的**人物画看一看，或者只是想象一下——一幅人物画中，手指的面积有多大？除非为了强调手部而加了特技，否则手指所占的面积是很小的；而大腿，在一幅正常的人物画中，是充满空间的。这里说的充满，当然并不是指完全占满空间，也不是上下左右一定没有空隙。


记住这种“充满”，现在思考一下 Flexbox。什么时候，我们会想到或者说需要弹性这件事情？答案是当我们需要“充满”一个容器的时候。带着这种思考，再回到人物画。手指只是在画面的特定位置，解决这种问题，我们可以简单地用 `position: absolute;` 或 `float: left;` 这些属性搞定。而大腿是“充满”画面的，当我们需要“充满”容器的时候，**弹性**就很重要！要解决这类问题，我们应该思考的就不是某个局部的空间，而是空间的分配。包括如何分配容器内的所有盒子，如果空间过大怎么办，如果空间过小怎么办；而在移动端，设备的屏幕尺寸有很多种，问题就变成了空间有时候大、有时候小怎么办。


读者应该已经能够想到，弹性盒模型就是为了更方便地解决这些问题而产生的。那么先看一看**非弹性**的盒子遇到这些场景会有哪些不便。


## 百分比网格

See the Pen [understanding-css-flexbox 1: percentage](https://codepen.io/AlphaBao/pen/rYpJMx/) by Alpha Bao ([@AlphaBao](https://codepen.io/AlphaBao)) on [CodePen](https://codepen.io).



上面是用浮动和百分比的方式写的横向网格，由于直接给其中一个设置了不同的高度，很明显，可以看出，四个格子的高度是不同的。如果格子的高度变化是由其中的内容引起，也会存在同样的问题。


另外，示例之中是四个格子，所以设置每个格子为 `width: 25%;` 就可以让它们横向充满父级容器，而且大小变化也没有影响。但如果需要渲染的数据是动态的，写成具体某个百分比显然就不行了。即元素个数变化时每个元素的百分比也需要变化，就需要修改 CSS。



这些问题都是因为这样的盒模型是没有“弹性”的，如果有弹性，就可以让布局按照我们希望的方式渲染。



## Flex 网格

```
.container {
	display: flex;
}
```

弹性盒模型带来了 Flexbox 布局，像上面这样，给充当 container 的盒子设置 `display: flex;` 就可以让它的子元素弹性排列，默认是横向的，因为 `flex-direction: row;` 是默认值，我们先不关心它。先看一下最常用的属性 `flex-grow`。



### flex-grow

See the Pen [understanding-css-flexbox 2: flex-grow](https://codepen.io/AlphaBao/pen/pdpKOj/) by Alpha Bao ([@AlphaBao](https://codepen.io/AlphaBao)) on [CodePen](https://codepen.io).



可以看到其中一个是 `flex-grow: 2;`，其他都是 1，意思是这些子元素将充满容器，它们将容器分成了若干份，每个 `flex-grow: 1;` 元素占据一份，`flex-grow: 2;` 的占两份，因为它的 `flex-grow` 值是其他元素的两倍。也就是说，`flex-grow` 决定子元素如何膨胀。在 Flexbox 的充满／填充策略中，`flex-grow` 影响的是元素膨胀到多大。注意这其实是如何分配父级容器空间的问题，而且是容器大小会改变的情形下，所以具体子元素的大小是取决于空间剩余情况的，并不是 `flex-grow` 越大，元素就一定会越大。



### flex-basis

再看 `flex-basis` 属性，它是指元素的初始大小，上一个例子中，只设置了 `flex-grow`，所以初始大小就是元素内容决定的，如果元素没有内容，大小就是零。



See the Pen [understanding-css-flexbox 3: flex-basis](https://codepen.io/AlphaBao/pen/ZavjGK/) by Alpha Bao ([@AlphaBao](https://codepen.io/AlphaBao)) on [CodePen](https://codepen.io).



`flex-basis` 的默认值是 `auto`，是指元素的大小（本文中指的是元素横向的长度，因为 `flex-direction` 默认值是 `row`，这决定了 main axis（主轴）是横向的，即容器的子元素横向排列）根据元素的长度属性或者由内容决定。可以是具体长度值也可以是百分比。



```
.c3 {
	width: 15em;
	flex-basis: auto;
}
```

此时初始宽度是 `15em`，也可以写为下面这样：



```
.c3 {
	flex-basis: 15em;
}
```

两种写法效果相同。



计算完初始大小，再根据容器空间剩余情况，继续完成“充满”容器这件事情。如果先只考虑空间还有剩余的情况，前面提到的 `flex-grow` 属性就开始起作用，使元素膨胀，直到充满容器。



### flex-shrink

前面考虑的都是空间还有剩余的情况，接下来考虑一下空间不足的情况。首先要弄清楚，具体怎样会导致空间不足。



See the Pen [understanding-css-flexbox 4: flex-shrink](https://codepen.io/AlphaBao/pen/OOzwQM/) by Alpha Bao ([@AlphaBao](https://codepen.io/AlphaBao)) on [CodePen](https://codepen.io).



可以看到，通过设置宽度或者由内容填充，此时可能导致空间不足。此时 `flex-shrink` 会起作用。首先计算初始大小，再考虑空间不足的情况，这时候根据 `flex-shrink` 的值决定如何收缩各个元素，数值越大，相对其他元素的收缩倍数就越大。`flex-shrink` 默认值是 `1`，即不改变这个属性值的情况下，空间不足时每个元素的收缩程度相同。如果改为 `0`，则不收缩。



### flex

flex 是上面三者的简写形式（这类写法的属性就是 Shorthand properties）。



```
.item {
	flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]
}
```

See the Pen [understanding-css-flexbox 5: flex](https://codepen.io/AlphaBao/pen/MOrBZV/) by Alpha Bao ([@AlphaBao](https://codepen.io/AlphaBao)) on [CodePen](https://codepen.io).



上面的每个子元素都设置了 `flex: 1 1 100px;`，即初始宽度值是 100px，如果空间剩余，每个元素平均分配，如果空间不足，每个元素同等程度的收缩。`flex: 0 1 auto;` 是 `flex` 属性的默认值，表示不膨胀（如果都不膨胀，那么有剩余空间也不会充满），收缩因数是 `1`，元素在主轴方向上的长度取决于长度值或者元素内容。



通过上面几个例子，可以看得出在大小会动态变化的父级容器里面，这种分配空间的策略优势很明显，这就是弹性带来的便利。`flex` 是实际开发中常用的写法，它的内容其实就是如何“充满”容器空间的策略，是 Flexbox 中最重要的部分。



## 结束语

本文只谈及了 `flex-grow` `flex-shrink` `flex-basis`，关于 Flexbox，还有很多内容。比如决定缠绕方式的 `flex-wrap`，它的效果类似 `float`，还有 `justify-content`，它决定元素在 main axis（主轴）方向的对齐方式，`align-self` 则决定 cross axis（垂直的交叉轴）方向（在例如主轴是水平方向，而各个元素具有不同的高度这类情况下起作用）。属性名有点乱，不过它们都是围绕**充满**和**弹性**扩展开的。只要结合对“充满”空间这件事情的想象，理解了**弹性**的含义，就弄清了目标与方法，应该能比较容易地学会并运用 Flexbox 了。



如上所见，本文所谈的，并不是大腿这件事情，如果你想看“真正的”大腿，可以读一读古龙的《多情剑客无情剑》、《午夜兰花》、《长生剑》以及《萧十一郎》。



## 扩展阅读
1.  [A Complete Guide to Flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
2.  [A Visual Guide to CSS3 Flexbox Properties](https://scotch.io/tutorials/a-visual-guide-to-css3-flexbox-properties)
