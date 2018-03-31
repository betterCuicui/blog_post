---
title: redis数据结构_跳跃表
date: 2017-06-14 17:58:48
tags: [php,linux,redis]
category: [REDIS]
---
 
 
[跳跃表原理简单版](http://www.cnblogs.com/thrillerz/p/4505550.html)
![](/public/image/redis/f26a06e9-481a-4c59-b0eb-bc506a50fc94.jpg)
<!--more-->
这个是跳表的简单实现图。redis中的跳表原理就是这个，只是复杂了点。
redis跳跃表是一种有序的数据结构，它通过每个节点中维持多个指向其他节点的指针，从而达到快速访问节点的目的。ZSET就是使用的跳跃表。而set是使用的hash。
 
## 1.1、跳表的数据结构
```
/*
* 跳跃表
*/
typedef struct zskiplist {
    // 表头节点和表尾节点
    struct zskiplistNode *header, *tail;
    // 表中节点的数量
    unsigned long length;
    // 表中层数最大的节点的层数
    int level;
} zskiplist;
```
 
## 1.2、跳表的节点数据结构
 
```
//跳跃表节点
typedef struct zskiplistNode {
    // 成员对象
    robj *obj;
    // 分值
    double score;
    // 后退指针
    struct zskiplistNode *backward;
    // 层
    struct zskiplistLevel {
        // 前进指针
        struct zskiplistNode *forward;
        // 跨度
        unsigned int span;
    } level[];
} zskiplistNode;
```
## 1.3、redis中跳表的示意图
![](http://1e-gallery.redisbook.com/_images/graphviz-e252c0a9575f171b9721162311df23889699cac9.png)
- L2代表zskiplistLevel[1],后面的线代表forward前进指针，span为指针线上的数字，代表跨度。
- BW代表指向前一个节点的指针。
- 1.0/2.0/3.0代表score。
- O1/O2/O3代表obj，是一个SDS结构体。
- score相同的按照对象在字典中的位置先后排序。
 
## 1.4、跳表的使用-插入数据
- 1、生成随机数，表示该节点的最高的level。这个随机数最高是32，切level越大，这个概率越低。
- 2、遍历跳表，遍历的方式从最高的level开始遍历，如上图所示。
- 3、遍历的同时，用数组记录自己的level[L]->forward下一个需要指向的节点
- 4、遍历的同时，用数组记录自己的rank[L],就是每一level的跨度和，这样能算出总的rank和每一个level的span。
- 5、创建新的节点，初始化数据，插入相应的位置。