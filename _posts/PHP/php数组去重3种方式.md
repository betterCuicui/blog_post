---
title: php数组去重3种方式
date: 2017-07-03 11:20:19
tags: [php]
category: [PHP]


---

在对数组进行去重的时候，我们第一想到的api就是array_unique。但是array_unique并不是最快的。
比如使用`array_key(array_flip($array))`或者使用`array_flip(array_flip($array))`。
虽然这三种效果都是一样的，但是效率，后面两种却比前面的快了特别多。这是为什么呢？

<!--more-->

## array_unique
我们先来讲一下array_unique这个api的实现原理。这个函数是实现原理跟[sort排序实现](http://bettercuicui.github.io/2017/08/03/php/php%20sort%E6%8E%92%E5%BA%8F%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86/)有很大的相同的地方。
- 1、分配内存，初始化一个指针数组，数组里面每一个值，都指向了php数组的hashTable的节点。
- 2、对这个指针数组中，每个指针指向的值进行排序，使用的快排。
- 3、对排序后的数组，相同的值进行删除操作。
- 4、free掉申请的指针数组。

**可以清楚的看到，array_unique这个函数的时间复杂度是O(n long n)，主要耗时在快排上，控件复杂度在O(n)。**

## array_flip
这个函数原本是用来把php数组的key和value互换位置的。那么它的原理是什么呢？
- 申请一个数组A。
- 遍历原来的hashtable底层链表，对每一个节点，都把节点的value赋值为数组A的key。节点的key赋值为数组A的value。
- 对重复的key进行覆盖写

**可以看出，这个api的时间复杂度是O(n)，空间复杂度为O(n)，但是当使用array_flip(array_flip($array))的时候，需要两次空间分配，浪费了空间**

## array_key
这个函数很简单，就是获取所有的数组的key，原理也狠简单
- 申请一个数组B
- 遍历原数组底层的链表，获取每一个节点的key，插入新的数组B.

**所以这个api的时间复杂度和空间复杂度都是O(n)**

## 区别
那么这三个方式，分别应该什么时候使用呢？
- 因为array_unique的空间复杂度是O(n),而其他的两种都是O(2n)。
- array_unique的时间复杂度是O(n long n),其他的都是O(n)。
- 其他两种相对于array_unique，应该是属于使用了空间换取了时间。

## 资料
- [【性能为王】从PHP源码剖析array_keys和array_unique# 【性能为王】从PHP源码剖析array_keys和array_unique](http://blog.csdn.net/lz610756247/article/details/51512918)