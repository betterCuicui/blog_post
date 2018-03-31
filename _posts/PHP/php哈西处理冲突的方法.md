---
title: php处理哈西冲突的方法
date: 2016-12-01
tags: [php]
category: [PHP]
---
前面介绍过[php的hash算法](http://bettercuicui.github.io/2016/12/01/php/php数组的hash算法/)是DJBX33A哈西算法。目前来说是不存在没有冲突的hash算法的，评价一个hash算法的好坏一般是看中数据分部的均匀程度与计算的效率。
既然有冲突，我们就要解决冲突。
<!--more-->
### 一、php使用拉链法
拉链法解决冲突的方式是：**把所有哈西值为同一个值的数据，链接在一个链表上**。若选定的散列表长度为m，则可将散列表定义为一个指针数组t[0.1....m-1]，该数组由m个头指针组成。所有hash结束后的为i的值，都插入到t[i]后面接着的**链表的尾部**。
 
### 二、散列表长度M值怎么定？拉链前干了什么？
假设`$arr = array('a'=>1,);`出现这样的代码，根据php的hash算法`hash = 5381;hash = hash*33+*str++;`得到  `5381*33+97 = 177670`,哈西值为177670。而这个数组中只有一个值，我们总不能创建一个177670大小的数组，然后t[177670] = 1?
#### 2.1 为了减少内存的消耗，php是这么处理的
这个是zend_hash.c中哈西表的初始化代码：
```
ZEND_API int _zend_hash_init(HashTable *ht, uint nSize, dtor_func_t pDestructor, zend_bool persistent ZEND_FILE_LINE_DC)
{
    uint i = 3;
 
    SET_INCONSISTENT(HT_OK);
 
    if (nSize >= 0x80000000) {
        /* prevent overflow */
        ht->nTableSize = 0x80000000;
    } else {
        while ((1U << i) < nSize) {
            i++;
        }
        ht->nTableSize = 1 << i;
    }
//省略初始化一些别的数据
......
    return SUCCESS;
}
```
从中我们可以看出：
 
- 1、php的hashTable大小最小的是2的3次方，也就是8；
- 2、php的hashTable大小最大是0x80000000（2^31）；
- 3、php的hashTable大小都是2的指数。比如说一个有10个元素的数组，那么hashTable大小就是16，也就是说M就是16；如果元素个数是20，M的大小就是32了。
- 4、如果存入的元素大于当前最大存储的个数，那么就会进行扩容处理了。每次扩容的大小为原来的2倍。
 
#### 2.2 拉链前需要先hash&tableMask
在zend_hash.c中，我们在对数的增删改查函数时，都可以看到这样一行代码：`nIndex = h & ht->nTableMask;`那么这样代码是什么意思呢？这行代码就是数据在进行hash算法后、拉链前进行的计算。
现在我们要存储7个元素，那么我们知道数组的容量应该是8了，也就是nTableMask（数组最大的下标,也就是M-1）应该是7，二进制是0111；
而假设我们现在需要存储一个元素9，那么（9 & 7 = 1）换成二进制是（1001 & 0111 = 0001），那么9就应该存储到t[1]的链表后。如图：
![](/public/image/hash1.jpg)
 
### 三、前方高能---hash冲突导致服务器攻击
先看例子：
 
```
<?php
set_time_limit(0);
header("Content-type: text/html; charset=utf-8");
$size = pow(2, 16);    //65536
$startTime = microtime(true);
$array = array();
for ($key = 0, $maxKey = ($size - 1) * $size; $key <= $maxKey; $key += $size) {
    $array[$key] = 0;
}
$endTime = microtime(true);
echo '插入 ', $size, ' 个恶意的元素需要 ', $endTime - $startTime, ' 秒', "<br/>";
 
$startTime = microtime(true);
$array = array();
for ($key = 0, $maxKey = $size - 1; $key <= $maxKey; ++$key) {
    $array[$key] = 0;
}
$endTime = microtime(true);
echo '插入 ', $size, ' 个普通元素需要 ', $endTime - $startTime, ' 秒';
exit;
```
 
在我机器上的执行结果是：
 
```
插入 65536 个恶意的元素需要 61.527000188828 秒
插入 65536 个普通元素需要 0.021000146865845 秒
```
 
这下问题出来了：**为什么同样插入32768个数据，消耗时间会相差这么多，差别还这么夸张？**
![](/public/image/hash2.jpg)
根据`nIndex = h & ht->nTableMask`这行代码，ht->nTableMask = 65536-1，换算成二进制是（0 1111 1111 1111 1111），接下来0为初始值，每次增加65536的大小，那么可以推测下面的计算：
 
```
0000 0000 0000 0000 0000 & 0 1111 1111 1111 1111 = 0
0001 0000 0000 0000 0000 & 0 1111 1111 1111 1111 = 0
0010 0000 0000 0000 0000 & 0 1111 1111 1111 1111 = 0
0011 0000 0000 0000 0000 & 0 1111 1111 1111 1111 = 0
0100 0000 0000 0000 0000 & 0 1111 1111 1111 1111 = 0
```
 
从上我们可以看出插入第一个数是0，会接在t[0]的链表，第二个数是65536，同样会链接在t[0]的链表尾部上；第三个数是`65536*2`，一样会链接在t[0]的链表尾部上。。。。。以此论推。而且，因为每次插入数据都是链接到链表的尾部上，所以第一个数需要遍历0个元素，第二个元素需要遍历1个元素。。。当插入第65536个数的时候，需要遍历65536个数。那么总共就进行`65536*65536/2=2147385345`次遍历。
 
**并且由于每次插入的数据都链接到了t[0]的链表上，那么这个hash表就退化成了链表**
这样一来, 如果数据量**足够大**, 那么就可以使得语言在计算, 查找, 插入的时候, 造成大量的CPU占用, 从而实现拒绝服务攻击.
 
PHP5.4是通过增加一个限制来尽量避免被此类攻击影响:
 
```
- max_input_vars - specifies how many GET/POST/COOKIE input variables may be
accepted. default value 1000.
```
 
其实刚开始我觉得，在插入数据的时候，可以有方法避免遍历链表的：
 
- 1、可以直接在t[i]链表的头插入；
- 2、t[i]记录当前链表的尾，然后直接在尾插入。
 
**后来通过看代码发现，每次插入都需要遍历链表，是为了防止用户插入相同的key值。**= . =