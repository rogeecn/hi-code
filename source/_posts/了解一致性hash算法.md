---
title: 了解一致性hash算法
date: 2018-02-11 00:00:00
tags: ["算法"]
abbrlink: consistent-hashing-theory-introduction
img: ""
comments: false
---

## 定义
一致性hash算法，在维基百科的定义是：

> Consistent hashing is a special kind of hashing such that when a hash table is resized, only K/n keys need to be remapped on average, where K is the number of keys, and n is the number of slots. In contrast, in most traditional hash tables, a change in the number of array slots causes nearly all keys to be remapped because the mapping between the keys and the slots is defined by a modular operation.

翻译过来的意思就是当hash表更新节点的数量时，只有k/n的关键字位置有变化，其他关键字的位置映射关系不变。与其他的hash算法比，其他的算法节点个数n变化后，更多的key关键字和节点的映射会发生变化。



## 使用场景

一致性hash主要用在路由中，对有状态的服务，根据key进行转发到对应的服务中。保证相同的key一直落到同一个服务器，当有服务节点增减时，只有少量(k/n)的请求位置是变化的。减少重新建立缓存或存储的成本。

## 原理实现

前提：

1. 每个请求的key范围[0,2^32)，一共有k个key;
2. 一共有N个节点，一般一个几点对应一个服务器。

### 常规实现

取key所映射的所有值最大空间（2^32）个，组成一个环，然后随机在这个环上落N个点，相邻的两个点形成一个左闭右开（关于左闭右开参考《聊聊左闭右开区间》)区间。共有N个区间。

对于每个key，一定只落在N个区间中的一个，它属于该区间所分配的节点。

当有服务节点增减时，会有区间新增或消失，平均只有k/N个key会受影响，变更属于的节点。

如下图，在插入nodeC之前，2、3、8key都属于nodeA，当插入nodeC后2、3归属C，属于B的节点不会改变。

![](http://oss.ipaoyun.com/blog/1-1512439915.png)

### 改进：增加虚节点

常规实现在实际应用中会遇到问题。当N的数量太少时，会导致N个节点所管辖的区间并不均匀。

既然是N的数量太少，那增加N的数量不就行了?正解，可以成倍地增加N的数量，一个实际的节点扩充为100倍的虚节点，每个key先查找属于哪个虚节点，再查看该虚节点属于那个实节点。

由于众多虚节点的引入，使每个实节点被分配到的key数量的差距变少。

![](http://oss.ipaoyun.com/blog/2-1512439932.png)

从图中可见，增加了nodeA和nodeD的虚节点后，把区间分得更细小，会使分布更均匀。还可以通过设置权值，让不同处理能力的实节点，处理不同量级的key。

### 实践经验

通过上面的讲解，可以熟悉一致性hash的算法，但是在实际使用中，还是有很多需要注意的地方。

### 如何加入虚节点

加入虚节点能够解决分布不均的问题，但是如何加入也是有技巧的。如果完全随机，就是撞大运编程。要利用搜索算法，加入节点时要检测，保证每个实节点的区间不能差异太大。必要时要回溯，剪枝，或者用启发性搜索。

### 节点配置同步

一个大系统，每个真实节点有1000个虚节点，一共1000个实节点，有1M条目数据。每当更新节点信息时，要保证快速更换、传递更新数据，而且要有检查功能。节点配置的同步、检测也会有很多细节问题。
