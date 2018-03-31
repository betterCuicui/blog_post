---
title: redis数据结构_整数集合
date: 2017-06-14 14:58:48
tags: [php,linux,redis]
category: [REDIS]
---
redis数据结构_整数集合
<!--more-->
## 第6章：整数集合
![](http://1e-gallery.redisbook.com/_images/graphviz-acf7fe010d7b09c5d2500c72eb555863e67ad74f.png)
```
redis 127.0.0.1:6688> sadd numbers 1 2 3 354 6 7 2
(integer) 6
redis 127.0.0.1:6688> OBJECT ENCODING numbers
"intset"
redis 127.0.0.1:6688> sadd numbers fds
(integer) 1
redis 127.0.0.1:6688> OBJECT ENCODING numbers
"hashtable"
```
当一个集合只是包含整数，并且值不多的时候，底层使用intset，否则用hashtable。
**总结：**
- 1、整数集合的底层实现为数组，这个数组以有序、无重复值的方式保存集合元素。
- 2、整数集合初始化的时候，存储的整数为int16_t，当存储的值有大于int16_t的时候，比如是int32_t,那么数组会重新申请一块更大的空间，并且把以前的数据复制进去，然后把新的数据插入。
- 3、每次插入数据的时候，会申请新空间，以便于存入所有的值。
- 4、每次删除数据的时候，会把所有的数据往前挤一个位置，然后把最后的一个位置free掉。
- 5、只会数据格式升级，不会降级。也就是说当数据格式是int32_t，把唯一的一个int32_t数据删除，剩下的都是int16_t，数组也不会降级。