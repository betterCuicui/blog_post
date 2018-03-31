---
title: redis数据结构_链表
date: 2017-06-14 17:58:48
tags: [php,linux,redis]
category: [REDIS]
---
 
链表提供了高效的节点重排能力，以及顺序性的节点访问方式，并且可以通过增删节点来灵活的调整链表的长度。被广泛用于实现redis各种功能，比如列表键、发布与订阅、慢查询、监视器等。
<!--more-->
 
## 1.1、链表和链表节点的实现
### 1.1.1、节点的结构
adlist.h/listNode
```
typedef struct listNode{
    //前置节点
    struct listNode *prev;
    //后置节点
    struct listNode *next;
    //节点的值
    void *value;
}listNode;
```
### 1.1.2、链表的结构
```
typedef struct list{
    //表头结点
    listNode *head;
    //表尾节点
    listNode *tail;
    //链表所包含的节点的数量
    unsigned long len;
    //节点值复制函数
    void *(*dup)(void *ptr);
    //节点值释放函数
    void (*free)(void *ptr);
    //节点值对比函数,判断链表节点保存值对输入值是否相等
    void (*match)(void *ptr,void *key);
}list;
```
![](/public/image/redis/0913d1c4-6fec-4109-abb0-f56f161c5eba.png)
**特性总结：**
- 1、双端。
- 2、无环。
- 3、含有头尾节点指针。
- 4、含有链表节点计数器。
- 5、多态。值为`void*`，可以保存任意数据类型。