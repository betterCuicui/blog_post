---
title: 将Linux命令的结果作为下一个命令的参数
date: 2016-06-15 16:14:58
tags: [linux]
category: [OTHERS]
---


[http://blog.csdn.net/permike/article/details/51957003](http://blog.csdn.net/permike/article/details/51957003)
转自这里
<!--more-->
**作用：**
- 、查询最新的日志信息
 
```
tail -f /home/map/odp_cater/log/antispam/`ls /home/map/odp_cater/log/antispam| grep antispam.log.201| tail -1`
```
 
### 一、符号：\` \`
名称：反引号，上分隔符
位置：反引号（\`）这个字符一般在键盘的左上角，数字1的左边，不要将其同单引号（’）混淆
作用：反引号括起来的字符串被shell解释为命令行，在执行时，shell首先执行该命令行，并以它的标准输出结果取代整个反引号（包括两个反引号）部分
 
使用：可以用于把结果作为多个参数之一的需求
举例：
 
```
$ echo `date`
Thu Mar 7 21:31:11 CST 2013
```
 
### 二、$()
效果同\` \`
 
### 三、命令：xargs
xargs是给命令传递参数的一个过滤器，也是组合多个命令的一个工具。它把一个数据流分割为一些足够小的块，以方便过滤器和命令进行处理。通常情况下，xargs从管道或者stdin中读取数据，但是它也能够从文件的输出中读取数据。xargs的默认命令是echo，这意味着通过管道传递给xargs的输入将会包含换行和空白，不过通过xargs的处理，换行和空白将被空格取代。
```
<code class="hljs bash" style="display:block; padding:10px; background-color:rgb(253,246,227); color:rgb(119,119,119); overflow-x:auto; line-height:1.4; word-wrap:normal">$ date | xargs <span class="hljs-built_in" style="color:rgb(38,139,210)">echo</span>
Thu Mar 7 21:47:12 CST 2013</code>
```
 
管道与xargs的区别：
 
*   管道是实现“将前面的标准输出作为后面的标准输入”
*   xargs是实现“将标准输入作为命令的参数”
 
### 四、find命令的-exec参数
 
xargs：通过缓冲方式并以前面命令行的输出作为参数，随后的命令调用该参数
若忽略 xargs 的 options 来看的话,
cm1 | xargs cm2
可以单纯看成: cm2 `cm1`
因此, find .... | xargs rm 也可作 rm `find ...` 来处理.
然而, 若 find 的结果太多, 可能会超过rm 可能接受的最大argument数量而失败.
xargs优点：由于是批处理的，所以执行效率比较高（通过缓冲方式）
xargs缺点：有可能由于参数数量过多（成千上万），导致后面的命令执行失败
若换成 find .... -exec   rm {} \; 的话,
因为rm 是" 逐个 " item 去处理的, 则无此忧虑
 
**参考：**
[如何将Linux命令的结果作为下一个命令的参数](http://www.cnblogs.com/eshizhan/archive/2011/11/30/2269325.html)
[Linux xargs命令](http://blog.csdn.net/sunboy_2050/article/details/7303501)
[管道命令和xargs的区别(经典解释)](http://blog.csdn.net/yongan1006/article/details/8134581)
[exec 与 xargs的区别](http://blog.csdn.net/offbye/article/details/7053069)