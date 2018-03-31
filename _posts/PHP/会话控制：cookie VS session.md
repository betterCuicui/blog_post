---
title: 会话控制：cookie VS session
date: 2016-04-03 20:20:50
tags: [php]
category: [PHP]
---
- 什么是会话？
我的理解中，tcp中，从三次握手到四次挥手，可以算一次会话；打开一个网页算一次会话；从登录网页到注销算一次会话。
- 什么是会话控制？
会话控制的控制的思想就是允许服务器`跟踪同一个客户端做出的连续请求`。
<!--more-->
 
#### http是无状态协议，那么怎么才能从一个页面跳转到另一个页面，但是信息保存呢？让服务器知道这是同一个用户呢？
 
##### 1、通过文件、数据库存储信息
eg：
a.php
```
<?php
$username = 'cuicui';
file_put_contents('global.txt',$username);
```
b.php:
```
<?php
file_get_contents('global.txt',$username);
echo $username;
```
缺点：太不方便了，文件的读取速度，全局，每个人都共享信息？；
##### 2、通过url的get或者header（）等重定向方式来传递信息
假设一个网页中有几千个url？用户信息假设有很多变量？特别麻烦
##### 3、会话级别 cookie session
同一个用户，同一个网站，共享自己的信息
 
## 一、cookie
#### 1、cookie简介：
- 什么是cookie
当用户访问服务器，服务器给用户设置cookie给用户，发送给浏览器，浏览器会存在内存中或者在特定文件夹下面建立cookie文件，记录你访问网站的信息：看你想把什么变变量设置给cookie 了。里面为二进制信息，我也看不懂。当你下回在访问网站的时候，会把网站所对应的cookie发送给服务器。
- cookie位置
比如chrome的cookie位置在内存中或者：`C:\Users\cuicui\AppData\Local\Google\Chrome\User Data\Default`，也可以在浏览器设置->高级设置->隐私->cookie中看到。
注意：如果浏览器上cookie太多，超过了系统所允许范围，浏览器也会自动对它进行删除。
- cookie内容
常用于保存用户名，密码，个性化设置，个人偏好记录等。
- cookie与http的关系
响应头往客户端设置cookie变量：setCookie函数
请求的头信息把cookie待到服务器
 
###### 注：服务端往客户端传文件，比较不安全，还好设置了只在特定位置传特定文件；很多网站偷偷搜集用户信息，可能就是通过cookie文件；
#### 2、cookie操作：
- 设置cookie
语法：`bool setcookie(string name,string value,int expire,string path,string domain,int secure);`
注：前两个是必须的，后面参数都是可选的；
```
setcookie('username','cuicui',time()+60*60*27);//保存一天
setcookie("username", "cuicui", time()+60*60*24,  "/test",  ".baidu.com",  1);
```
- cookie参数解释
name->名字
value->值
exprice->有效时间
path->范围->服务器的有效路径，设置为‘/’表示这个域中所有的数组都可以访问
domain->域名
secure->指定cookie只能通过https传送（0或者1）；
- 读取cookie
`$_cookie['name']`
- 删除cookie
```
1、setcookie（‘name’）；
2、setcookie（‘name’，‘’，time()-1)）；
```
 
## 二、session
#### 1、session简介：
session跟cookie相似，都是用来存储使用者的相关资料，存储方式为session_id=>contents,当用户访问web的时候，会生出一个session_id，并且把session_id存储在cookie中，等下回用户再次访问这个网页的时候，会把客户端cookie中的session_id返回给服务器，服务器通过session_id去寻找contents。
#### 3、session操作
- session的声明（使用session都必须先有声明）
`bool session_start(void);`
注意：session_start()函数之前不能有任何输出
- 存储session信息
`$_SESSION['views']=1;`
- 删除服务器端保留session信息的文件
`bool session_destroy(void)`
- 删除内存中由Session数组保存的变量
`unset($_SESSION[‘键名’])`
- 清除所有变量可以使用 `$_SESSION=array()`
- 如果session是基于Cookie的，那么我们还需要删除客户端保留的cookie文件
## 三、cookie VS session
 
### 区别
#### 1、存储位置区别
- cookie 存放在客户端浏览器中，
- session 存放在服务器上
 
#### 2、安全性
- cookie不安全，别人可以分析存放在本地的cookie并且进行cookie欺骗。
- session相对安全，因为它存放在服务器，用户是分析不到的
 
#### 3、性能上
- session一般保存在服务器内存中，当用户增多的时候，占用内存太大，所以把session设置保存在数据库中。
- cookie因为保存在客户端，则不需要考虑性能。
 
#### 4、保存数据大小
单个cookie保存的数据不能超过4k，很多浏览器都限制一个站点最多保存20个cookie。
 
### 联系
session把session_id存放在cookie中，但是如果客户端禁用了cookie则存放在url中。