---
title: php数组及遍历
date: 2017-01-08
tags: [php]
category: [PHP]
---
 
前面我们已经介绍了php数组中的hash算法以及拉链法。现在我们来深入hashTable。
<!--more-->
### 首先我们来看一下HashTable的结构定义：
```
typedef struct _hashtable {
    uint nTableSize;  //哈西数组的大小
    uint nTableMask;  //nTableSize-1,哈西数组的最大下标
    uint nNumOfElements;  //当前元素的个数。count()函数会直接返回这个值
    ulong nNextFreeElement;  //下一个空闲数字索引的位置
    Bucket *pInternalPointer;  //当前遍历的位置指针，会被reset, current这些遍历函数使用
    Bucket *pListHead;  //双向链表的头
    Bucket *pListTail;  //双向链表的尾巴
    Bucket **arBuckets;  //实际的存储容器，一个存储Bucket *的数组。
    dtor_func_t pDestructor;  //析构函数指针
    zend_bool persistent;
    unsigned char nApplyCount;  //循环遍历保护
    zend_bool bApplyProtection;
#if ZEND_DEBUG
    int inconsistent;
#endif
} HashTable;
```
可以看出：
- 1、nTableMask的作用就是在哈西算法后，拉链法前执行`nIndex = h & ht->nTableMask;`，来确定新增的值所对应的数组的下标。
比如`$arr=arr{'foo',};`，因为数组最小值是8，所以nTableMask在这里应该是7，'foo'通过哈西算法得到的值是193491849。而他对应的数组的下标是193491849&7=1。这样做为了把大的值一样映射到nTableSize的空间中。
- 2、nNumOfElements是count或者sizeof返回值。说明这两个函数都不是通过计算返回的。
- 3、nNextFreeElement指明了下一个空余元素的位置。比如说
例子1：
 
```
$arr[0] = 'a';
$arr[3] = 'b';
$arr[] = 'c';
$arr[5] = 'd';
foreach($arr as $k=>$v) {
    var_dump($k.'=>'.$v);
}
 
/*结果为
0=>a
3=>b
4=>c
5=>d
*/
```
 
```
$arr['q'] = 'a';
$arr['w'] = 'b';
$arr[] = 'c';
$arr['e'] = 'd';
foreach($arr as $k=>$v) {
    var_dump($k.'=>'.$v);
}
/*结果为
q=>a
w=>b
0=>c
e=>d
*/
```
 
### Bucket结构
```
typedef struct bucket {
    ulong h;  //哈西值，没有经过index&nTableMask得到的值
    uint nKeyLength;  //字符索引的长度
    void *pData;  //数据
    void *pDataPtr;  //数据指针
    struct bucket *pListNext;  //下一个元素,用于线性遍历
    struct bucket *pListLast;  //上一个元素，用于线性遍历
    struct bucket *pNext;  //处于同一个拉链中的下一个元素
    struct bucket *pLast;  //处于同一拉链中的上一个元素
    const char *arKey;  //strKey值。字符串key
} Bucket;
```
 
从中我们可以知道：
- 1、h是元素的Hash值,对于数字索引的元素,h为直接索引值(通过nKeyLength=0来表示是数字索引).
- 2、而对于字符串索引来说, 索引值保存在arKey中, 索引的长度保存在nKeyLength中.
- 3、在Bucket中，实际的数据是保存在pData指针指向的内存块中，通常这个内存块是系统另外分配的。但有一种情况例外，就是当Bucket保存 的数据是一个指针时，HashTable将不会另外请求系统分配空间来保存这个指针，而是直接将该指针保存到pDataPtr中，然后再将pData指向本结构成员的地址。这样可以提高效率，减少内存碎片。由此我们可以看到PHP HashTable设计的精妙之处。如果Bucket中的数据不是一个指针，pDataPtr为NULL。
 
**从这两个结构体，我们可以发现hashTable是由1个数组和2个双向链表来实现的。数组是用于查找哈西key值，其中一个双向链表用于遍历数据，其中一个双向链表是拉链，用于防止散列冲突的。**
 
### 接下来我们用数据来模拟一下：
假设现在有php代码：
 
- 1、第一步骤
 
```
$arr[0]=1;
```
 
![](/public/image/php数组及遍历1.jpg)
**创建一个哈西数组，数字能装8个链表，遍历的链表（红线）头尾巴都指向了最新申请的节点（0，1），然后数组0后面的拉链链表（蓝线）也指向了（0，1）；**
 
- 2、第二步
 
```
$arr[0]=1;
$arr[6]=1;
```
 
![](/public/image/php数组及遍历2.jpg)
**新建一个节点（6，1），数组6后面的拉链链表（蓝线）也指向了（6，1），遍历的链表头不变，尾巴指向了（6，1），节点（0，1）也有遍历指针指向了（6，1）**
 
- 3、第三步
 
```
$arr[0]=1;
$arr[6]=1;
$arr[8]=1;
```
![](/public/image/php数组及遍历3.jpg)
**新建一个节点（8，1），数组0后面的拉链链表（蓝线）也指向了（8，1），（8，1）后面的蓝色拉链指向了（0，1）。遍历的链表头不变，尾巴指向了（8，1），节点（6，1）也有遍历指针指向了（8，1）**
 
### 最后一张大图，可以很清楚的知道步骤
![](http://www.nowamagic.net/librarys/images/201303/2013_03_22_01.jpg)