---
title: redis单进程单线程优点
date: 2017-07-04 19:32:48
tags: [redis]
category: [REDIS]
---

其实redis并不是只有单进程单线程的。比如bgsave命令，就会fork一个新的进程，来把内存中的数据存储到本地。
我们所说的redis单进程单线程是指一般的情况下，它的整体架构是单线程单进程的。
<!--more-->

## 一、redis为什么快
所以redis快主要是因为三个原因：
- 完全基于内存
- 数据结构简单，对数据结构操作简单
- 使用了I/O复用模型
![](http://1e-gallery.redisbook.com/_images/graphviz-03e1d92303e2714abb2f8d719ef1546a9054e8ef.png)
如上图可以很清楚的发现，redis整体架构是一个死循环，这个死循环里面，先执行文件事件，再处理时间事件。
而且，redis的文件事件处理器是由多个i/o复用可以选择
![](http://1e-gallery.redisbook.com/_images/graphviz-883af630a14f2796161bbd33a4e7e2824e01d478.png)
虽然memcached也是用I/O复用，但memcached直接使用的libevent，而redis自己完成了一个非常轻量级的对select、epoll、evport、kqueue这些通用的接口的实现。在不同的系统调用选用适合的接口，linux下默认是epoll。因为Libevent比较重更通用代码量也就很庞大，拥有很多Redis用不上的功能，Redis为了追求“轻巧”并且去除依赖，就选择自己去封装了一套。

## 二、单线程单进程的优点
- 不用考虑各种锁的问题，不存在抢占资源，不需要加锁、解锁的操作，更不会担心产生死锁。
- 代码清晰，处理逻辑简单
- 不存在各种切换进程、线程而造成CPU的浪费