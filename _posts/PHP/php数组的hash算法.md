---
title: php数组的hash算法
date: 2016-12-01
tags: [php]
category: [PHP]
---
php的数组是万能的，能够实现一切的常用的数据结构：vector、双向单向链表、map等都不在话下。我觉得php的核心就是数组，而我们知道php的数组是基于hash实现的，那么hash就是php的核心了。
<!--more-->
#### 一 、DJBX33A哈西算法
DJBX33A (Daniel J. Bernstein, Times 33 with Addition)这个hash算法目前是特别普遍的，除了php、Apache, Perl和Berkeley DB等都使用这个哈西算法。对于字符串来说，这是目前最好的哈西算法，因为效率特别高，冲突小，分部均匀。
```
unsigned long DJBX33A(char const *str, int len)
{
    unsigned long  hash = 0;
    for (int i = 0; i < len; i++) {
        hash = hash *33 + (unsigned long) str[i]; //核心
    }
    return hash;
}
```
**为什么是乘33呢？而不是别的数字？因为作者使用了大量的数字，最后在哈西算法的分布与计算效率均衡上发现33是最好的.**
 
#### 二、php改进版哈西算法
我查看的是php-5.6.28版本，hash的源码在php-5.6.28/Zend/zend_hash.h。下面代码贴出来。
```
static inline ulong zend_inline_hash_func(const char *arKey, uint nKeyLength)
{
    register ulong hash = 5381;
 
    /* variant with the hash unrolled eight times */
    for (; nKeyLength >= 8; nKeyLength -= 8) {
        hash = ((hash << 5) + hash) + *arKey++;
        hash = ((hash << 5) + hash) + *arKey++;
        hash = ((hash << 5) + hash) + *arKey++;
        hash = ((hash << 5) + hash) + *arKey++;
        hash = ((hash << 5) + hash) + *arKey++;
        hash = ((hash << 5) + hash) + *arKey++;
        hash = ((hash << 5) + hash) + *arKey++;
        hash = ((hash << 5) + hash) + *arKey++;
    }
    switch (nKeyLength) {
        case 7: hash = ((hash << 5) + hash) + *arKey++; /* fallthrough... */
        case 6: hash = ((hash << 5) + hash) + *arKey++; /* fallthrough... */
        case 5: hash = ((hash << 5) + hash) + *arKey++; /* fallthrough... */
        case 4: hash = ((hash << 5) + hash) + *arKey++; /* fallthrough... */
        case 3: hash = ((hash << 5) + hash) + *arKey++; /* fallthrough... */
        case 2: hash = ((hash << 5) + hash) + *arKey++; /* fallthrough... */
        case 1: hash = ((hash << 5) + hash) + *arKey++; break;
        case 0: break;
EMPTY_SWITCH_DEFAULT_CASE()
    }
    return hash;
}
```
 
我们可以看出php源码中对原始的hash算法还是做了改进的：
- `hash = ((hash << 5) + hash) + *arKey++;`感觉改进最好的还是这行代码。把乘法改为了 << 。毕竟位操作的效率是大大超过乘法的。
- hash初始值由原来的0变为5381，根据鸟哥的意思是：这个值能提供更好的分类。
- 为什么要每次写8行`hash = ((hash << 5) + hash) + *arKey++;`。我的理解是一般当key的字符串不会超过8个字符，这样的话就可以避免for循环多次，提高效率。也有人说是考虑字节的对其，这个我目前还没有理解。