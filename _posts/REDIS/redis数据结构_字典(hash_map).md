---
title: redis数据结构_字典(hash_map)
date: 2017-06-14 18:58:48
tags: [php,linux,redis]
category: [REDIS]
---
 
- 首先，声明一点，这里的字典，并不是字典树。而是关联数组、映射（map），是一种保持键值对的抽象数据结构。
- 除了用来表示数据库，字典还是哈希键的底层实现之一。
<!--more-->
 
## 1.1、字典的实现
redis的字典使用哈希表作为底层实现。
### 1.1.1、哈希表
dict.h/dictht
```
typedef struct dictht{
    //哈希表数组
    dictEntry **table;
    //哈希表大小
    unsigned long size;
    //哈希表大小，用于计算索引值
    unsigned long sizemask;
    //该哈希表已有节点的数量
    unsigned long used;
} dictht;
```
![](/public/image/redis/487f672f-0f80-47d3-a9c5-10478804b4f8.png)
### 1.1.2、哈希表节点
```
typedef struct dictEntry{
    //键
    void *key;
    //值
    union{
        void *val;
        uint64_t u64; //typedef unsigned long long uint64_t;
        int64_t s64; //typedef long long int64_t;
    }v;
    //指向下一个哈希表节点，形成链表
    struct dictEntry *next;
} dictEntry;
```
![](/public/image/redis/8fd099c4-1f56-4eb9-affe-a7631e31da6b.png)
 
### 1.1.3、字典
dict.h/dict
```
typedef struct dict{
    //类型特定函数
    dictType *type;
    //私有数据
    void *privdata;
    //哈希表
    dictht ht[2];
    //rehash 索引,当rehash不再进行的时候，值为-1
    int trehashidx;
} dict;
```
- 1、type属性是一个指向dictType结构的指针，每个dictType结构保存了一簇用于操作特定类型键值对的函数。
- 2、privdata属性保存需要传给那些类型特定函数的可选参数。
 
```
typedef struct dictType{
    //计算哈希值的函数
    unsigned int (*hashFunction) (const void *key);
    //复制键的函数
    void *(*keyDup) (void *privdata, const void *key);
    //复制值的函数
    void *(*valDup) (void *privdata,const viod *obj);
    //对比键的函数
    int (*keyCompare) (void *privdata, const void *key1, const void *key2);
    //销毁值的函数
    void (*valDestructor) (void *privdata, void *obj);
} dictType;
```
- 3、ht属性中，ht[0]是字典使用的哈希表，ht[1]哈希表会是对ht[0]进行rehash的时候使用。
- 4、如果ht[0]没有在进行rehash，那么trehashidx为-1。
![](/public/image/redis/d128249f-01df-4208-802b-17077ef37822.png)
 
## 1.2、哈希算法
redis中使用的哈希算法是MurmurHash2。这个算法nginx也是使用这个。而PHP则是使用DJB算法。
```
unsigned int murMurHash(const void *key, int len)
    {
        const unsigned int m = 0x5bd1e995;
        const int r = 24;
        const int seed = 97;
        unsigned int h = seed ^ len;
        // Mix 4 bytes at a time into the hash
        const unsigned char *data = (const unsigned char *)key;
        while(len >= 4)
        {
            unsigned int k = *(unsigned int *)data;
            k *= m;
            k ^= k >> r;
            k *= m;
            h *= m;
            h ^= k;
            data += 4;
            len -= 4;
        }
        // Handle the last few bytes of the input array
        switch(len)
        {
            case 3: h ^= data[2] << 16;
            case 2: h ^= data[1] << 8;
            case 1: h ^= data[0];
            h *= m;
        };
        // Do a few final mixes of the hash to ensure the last few
        // bytes are well-incorporated.
        h ^= h >> 13;
        h *= m;
        h ^= h >> 15;
        return h;
    }
```
另外，当你的key经过hash得到的值一样需要与sizemask进行&操作的。比如插入k0->vo
```
//使用hash算法计算k0的hash值
hash = dict->type->hashFunction(k0);
//使用哈希表的sizemask属性和哈希值，计算索引值
//假设hash为8，ht[0]->sizemask为3；
index = hash & dict->ht[0]->sizemask = 8 & 3 = 0;
```
最终k0->vo键值对应该放置在数组索引0后。
![](/public/image/redis/aa22e418-b274-48b0-aab9-8b28bec0264c.png)
 
## 1.3、解决键冲突
使用拉链法来解决冲突，当有冲突的时候，把这个节点放在链表的头结点。
 
## 1.4、rehash
当节点的数量跟table的sizemask比例不均衡的时候，需要通过rehash来实现对哈希表大小的拓展或者收缩。
##### 1.4.1、rehash步骤如下:
- 1、为ht[1]分配空间。这个根据ht[0].used的大小来分配。如果是拓展操作，那么分配的大小是第一个大于或者等于`ht[0].used*2`的 2^n （2 的 n 次方幂）；如果是收缩操作，那么分配的大小是第一个大于或者等于ht[0].used的2^n；比如ht[0].used=5，那么拓展的时候，给ht[1]分配的大小为`2^n（2^n>=5*2）`，所以空间为2^4=16。如果是收缩操作，那么分配的空间为`2^n（2^n>=5）`，所以空间为2^3=8；
- 2、将ht[0]上的所有的k->v重新hash到ht[1]上；
- 3、把ht[0]的空间free，然后把ht[0] = ht[1]，ht[1]重新创建一个空hash表，用于下次rehash。
 
### 1.4.2、rehash时机
首先我们需要知道，哈希表的负载因子`load_factor=th[0].used/ht[0].size`;
- 1、服务器目前没有在执行 BGSAVE 命令或者 BGREWRITEAOF 命令(数据持久化)， 并且哈希表的负载因子大于等于 `1` ，进行扩展操作；
- 2、服务器目前正在执行 BGSAVE 命令或者 BGREWRITEAOF 命令(数据持久化)， 并且哈希表的负载因子大于等于 `5` ，进行扩展操作；
- 3、当哈希表负载因子小于0.1，进行收缩操作。
 
## 1.5、渐进式rehash
当需要扩展或者收缩的时候，并不是一次性rehash的，而是渐进性的rehash。就是在每次对字典的增删改查的时候，rehash一个拉链的链表。分而治之，避免集中rehash带来的庞大的计算量。
### 1.5.1、具体步骤
- 1、为ht[1]分配空间，让字典同时持有ht[0]和ht[1]两个hash表。
- 2、字典中维持一个索引计数器rehashindex，初始值为0。
- 3、在rehash期间，你每次对字典的增删改查操作，都会导致程序把ht[0]->table上的一条非空的链表上所有的键值对重新hash到ht[1]上面。结束后rehashindex值为该条链表对应的下标。
- 4、重复第3条步骤，当`ht[0].used=0`的时候，代表rehash已经全部结束。这个时候，rehashindex置为-1。
 
### 1.5.2、rehash期间的哈希表操作
- 因为在渐进式rehash的过程中，字典会同时使用ht[0]和ht[1]两个表，所以在进行删除、修改、更新的时候，也会在两个表上进行。比如查找k->v的时候，会先查询ht[0]，如果ht[0]没有查到，就继续查ht[1]。反正时间复杂度也是O(1)；
- 如果增加键值对的时候，会直接添加到ht[1]中。