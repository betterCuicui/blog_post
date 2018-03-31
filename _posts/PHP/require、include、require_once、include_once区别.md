---
title: require、include、require_once、include_once区别
date: 2016-05-04
tags: [php]
category: [PHP]
---
一直对这几个东西比较模糊，现在来系统总结一下，
而且网上很多的博客对这两个东西都写错了，比如返回值和是否有条件包含
<!--more-->
### 0、首先提醒一下，包含文件的时候，最好用全路径，能不用相对路径就不用，因为我在这个上面吃大亏了。
eg:假设a.php在./a/;   b.php在./a/;   c.php在./下
```
//a.php
<?php
echo "aaa";
?>
//b.php
<?php
require "./a.php";
?>
//c.php
<?php
require "./a/b.php";
?>
```
##### 如果运行c.php会输出aaa吗？no，因为不管require与include都是替代代码，所以c编译后会变成
```
<?php
require "./a.php";
?>
```
##### 但是当想进一步去require "./a.php"的时候却发现c的当前目录是./，但是跟他同级目录下，根本没有a.php。
所以就不会输出aaa了。但是如果相对路径换成全局路径就不会出现这样的情况了。。。
### 一、require 与 include区别
#### 1、加载失败处理方式相同
- require 失败后直接报错，中断程序
- include 失败后还会继续处理后面的程序
eg:假设不存在test1.php这个程序；
 
```
<?php
  require("./test1.php");
  echo "require<br/>";
?>
则报错，中断程序！！！
 
<?php
  include("./test1.php");
  echo "include<br/>";
?>
程序输出include！！！
```
### 二、require 与 include相同
#### 1、都是有条件包含
eg: aaa.php为`echo "aaa";`
```
if(false){
include "aaa.php";
}
if(false){
require "aaa.php";
}
```
##### 运行文件不会有任何输出，网上说require是无条件的，这里推翻了，所以这点网上错了
#### 2、都有返回值
```
//a1.php
<?php
return "aaa<br/>";
?>
 
//a2.php
<?php
$a = "aaa<br/>";
?>
 
//b.php
<?php
echo include "a1.php";
echo require "a1.php";
echo include "a2.php";
echo require "a2.php";
?>
```
输出
```
aaa
aaa
1
1
```
##### 说明，如果包含的文件里面有return则返回return的部分，否则返回1,
当然include有返回0的时候，就是文件不存在的时候，但是require不会返回0，因为文件不存在就程序崩溃啦...
#### 3、不加括号效率高
eg:
```
<?php
$start_time=microtime(true);
include("./test.php");
$end_time=microtime(true);//获取程序执行结束的时间
$total1=$end_time-$start_time; //计算差值
echo "<br/>此php文件中代码执行了{$total1}秒";
$start_time=microtime(true);
include "./test.php";
$end_time=microtime(true);//获取程序执行结束的时间
$total2=$end_time-$start_time; //计算差值
echo "<br/>此php文件中代码执行了{$total2}秒";
?>
```
结果total1>total2。。。所以这个是正解。
### 三、include与include_once（require与require_once）的区别
感觉平时都用include_once或者require_once比较好，可能是因为我从c++转过来的，
很重要的一点就是要保证文件只是包含一次。但是不知道为什么网上别人写代码都一般不用带_once的函数。