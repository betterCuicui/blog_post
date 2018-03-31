---
title:  php sort排序实现原理
date: 2017-08-02 10:46:19
tags: [php,sort]
category: [PHP]
---


php有很多的排序函数，比如普通的sort函数，倒序的rsort函数，还有按照键来排序的ksort、krsort函数，或者自定义比较函数的usort、ursort。这些在`/ext/standard/php_array.h`里面都有定义。
```
PHP_FUNCTION(ksort);
PHP_FUNCTION(krsort);
PHP_FUNCTION(natsort);
PHP_FUNCTION(natcasesort);
PHP_FUNCTION(asort);
PHP_FUNCTION(arsort);
PHP_FUNCTION(sort);
PHP_FUNCTION(rsort);
PHP_FUNCTION(usort);
PHP_FUNCTION(uasort);
PHP_FUNCTION(uksort);
```
现在我们拿sort函数举例
<!--more-->
## 一、sort函数介绍
```
bool sort ( array &$array [, int $sort_flags = SORT_REGULAR ] )
```
其中的sort_flags为可选。规定如何比较数组的元素/项目。可能的值：
*   0 = SORT_REGULAR - 默认。把每一项按常规顺序排列（Standard ASCII，不改变类型）
*   1 = SORT_NUMERIC - 把每一项作为数字来处理。
*   2 = SORT_STRING - 把每一项作为字符串来处理。
*   3 = SORT_LOCALE_STRING - 把每一项作为字符串来处理，基于当前区域设置（可通过 setlocale() 进行更改）。
*   4 = SORT_NATURAL - 把每一项作为字符串来处理，使用类似 natsort() 的自然排序。
*   5 = SORT_FLAG_CASE - 可以结合（按位或）SORT_STRING 或 SORT_NATURAL 对字符串进行排序，不区分大小写。

##### 可以查看sort在php源码的实现
```
PHP_FUNCTION(sort)
{
        zval *array;
        long sort_type = PHP_SORT_REGULAR;

        if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "a|l", &array, &sort_type) == FAILURE) {
            RETURN_FALSE;
        }

        php_set_compare_func(sort_type TSRMLS_CC);

        if (zend_hash_sort(Z_ARRVAL_P(array), zend_qsort, php_array_data_compare, 1 TSRMLS_CC) == FAILURE) {
            RETURN_FALSE;
        }
        RETURN_TRUE;
}
```

从源码可以看出：
- 先通过zend_parse_parameters进行参数或者的检验、转换等操作
- 再通过php_set_compare_func设置比较函数
- 最后通过zend_hash_sort进行排序

## 二、php_set_compare_func函数
sort_flags参数是会影响排序的结果。这个是通过php_set_compare_func函数来实现的：
```
static  void  php_set_compare_func(int sort_type TSRMLS_DC) /* {{{ */
{
    switch (sort_type &  ~PHP_SORT_FLAG_CASE) {
        case PHP_SORT_NUMERIC:
            ARRAYG(compare_func) = numeric_compare_function;
            break;
        case PHP_SORT_STRING:
            ARRAYG(compare_func) = sort_type & PHP_SORT_FLAG_CASE ? string_case_compare_function : string_compare_function;
            break;
        case PHP_SORT_NATURAL:
            ARRAYG(compare_func) = sort_type & PHP_SORT_FLAG_CASE ? string_natural_case_compare_function : string_natural_compare_function;
            break;
#if  HAVE_STRCOLL
        case PHP_SORT_LOCALE_STRING:
            ARRAYG(compare_func) = string_locale_compare_function;
            break;
#endif
        case PHP_SORT_REGULAR:
        default:
            ARRAYG(compare_func) = compare_function;
            break;
    }
}
```
**可以看到，不同的sort_flags会对应不同的对比函数。**

## 三、zend_hash_sort函数
这个函数的步骤如下：
#### 3.1、申请空间，初始化数组
```
arTmp = (Bucket **) pemalloc(ht->nNumOfElements  *  sizeof(Bucket *), ht->persistent);
p = ht->pListHead;
i =  0;
while (p) {
    arTmp[i] = p;
    p = p->pListNext;
    i++;
}
```
从上面可以看出，首先申请hashtable元素个数的数组。数组的每个元素里面装的是指针，指向ht中的节点。
#### 3.2、正式排序
```
(*sort_func)((void  *) arTmp, i, sizeof(Bucket *), compar TSRMLS_CC);
```
终于开始正式排序了。而排序的函数是一个指针，指向的是zend_qsort函数，所以我们继续跟踪
```
ZEND_API void  zend_qsort(void  *base, size_t nmemb, size_t siz, compare_func_t compare TSRMLS_DC)
{
    zend_qsort_r(base, nmemb, siz, (compare_r_func_t)compare, NULL TSRMLS_CC);
}
```
- zend_qsort_r函数主要是使用了快速排序来实现的。并且是非递归的。
- 重写了_zend_qsort_swap交换函数。这个函数是用来替换两个值的位置，因为通用性更好。

#### 3.3、排序后
```
ht->pListHead  = arTmp[0];
ht->pListTail  =  NULL;
ht->pInternalPointer  = ht->pListHead;
arTmp[0]->pListLast  =  NULL;
if (i >  1) {
    arTmp[0]->pListNext  = arTmp[1];
    for (j =  1; j < i-1; j++) {
        arTmp[j]->pListLast  = arTmp[j-1];
        arTmp[j]->pListNext  = arTmp[j+1];
    }
    arTmp[j]->pListLast  = arTmp[j-1];
    arTmp[j]->pListNext  =  NULL;
} else {
    arTmp[0]->pListNext  =  NULL;
}
ht->pListTail  = arTmp[i-1];
```
排序完后很容易看出，只是把数组里面的值，重新使用双向链表连接起来了。

整体的流程就是：
![](/public/image/php/sort/242206024998550.png)
排序后：
![](/public/image/php/sort/242206100624773.png)

## 三、总结
- php数组使用的排序算法是非递归的快排，时间复杂度是O（n*lgn）
- php排序需要额外的空间，空间复杂度是O(n)，所以需要避免对大数组进行排序

下面贴出排序算法：
```
ZEND_API void  zend_qsort_r(void  *base, size_t nmemb, size_t siz, compare_r_func_t compare, void  *arg TSRMLS_DC)
{
    void  *begin_stack[QSORT_STACK_SIZE];
    void  *end_stack[QSORT_STACK_SIZE];
    register  char  *begin;
    register  char  *end;
    register  char  *seg1;
    register  char  *seg2;
    register  char  *seg2p;
    register  int loop;
    uint offset;

    begin_stack[0] = (char  *) base;
    end_stack[0] = (char  *) base + ((nmemb -  1) * siz);

    for (loop =  0; loop >=  0; --loop) {
        begin = begin_stack[loop];
        end = end_stack[loop];

        while (begin < end) {
            offset = (end - begin) >>  1;
            _zend_qsort_swap(begin, begin + (offset - (offset % siz)), siz);

            seg1 = begin + siz;
            seg2 = end;

            while (1) {
                for (; seg1 < seg2 &&  compare(begin, seg1 TSRMLS_CC, arg) >  0;
                 seg1 += siz);

                for (; seg2 >= seg1 &&  compare(seg2, begin TSRMLS_CC, arg) >  0;
                 seg2 -= siz);
                if (seg1 >= seg2)
                    break;
                _zend_qsort_swap(seg1, seg2, siz);

                seg1 += siz;
                seg2 -= siz;
            }

            _zend_qsort_swap(begin, seg2, siz);

            seg2p = seg2;
            if ((seg2p - begin) <= (end - seg2p)) {
                if ((seg2p + siz) < end) {
                    begin_stack[loop] = seg2p + siz;
                    end_stack[loop++] = end;
                }
                end = seg2p - siz;
            }
            else {
                if ((seg2p - siz) > begin) {
                    begin_stack[loop] = begin;
                    end_stack[loop++] = seg2p - siz;
                }
                begin = seg2p + siz;
            }
        }
    }
}
```