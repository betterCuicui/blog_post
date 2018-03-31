---
title: cgi、fastcgi、php-cgi、php-fpm
date: 2017-08-02 10:46:19
tags: [php,cgi,fastcgi,php-fpm,php-cgi]
category: [PHP]
---
总结php,cgi,fastcgi,php-fpm,php-cgi之间的原理、区别、特点。
<!--more-->

## 一、CGI
### 1.1、cgi介绍
**通用网关接口**（**C**ommon **G**ateway **I**nterface/**CGI**）是一种重要的互联网技术，可以让一个客户端，从网页浏览器向执行在网络服务器上的程序请求数据。CGI描述了服务器和请求处理程序之间传输数据的一种标准。

### 1.2、工作原理
- 1.浏览器通过HTML表单或超链接请求指向一个CGI应用程序的URL。
- 2.服务器收发到请求。
- 3.服务器执行指定CGI应用程序。
- 4.CGI应用程序执行所需要的操作，通常是基于浏览者输入的内容。(url、header、post、get数据)
- 5.CGI应用程序把结果格式化为网络服务器和浏览器能够理解的文档（通常是HTML网页）。
- 6.网络服务器把结果返回到浏览器中。

### 1.3、特点
- webserver与具体程序处理独立出来，结构清晰、可控性强、耦合度高；
- 在高并发的情况下，cgi每次开启进程会成为服务器的大负担。性能会特别底下

## 二、FastCGI
### 2.1、FastCGI介绍
FastCGI像是一个常驻(long-live)型的CGI，它可以一直执行着，只要激活后，不会每次都要花费时间去fork一次(这是CGI最为人诟病的fork-and-execute 模式)。它还支持分布式的运算, 即 FastCGI 程序可以在网站服务器以外的主机上执行并且接受来自其它网站服务器来的请求(基于TCP)。

### 2.2、FastCGI架构
- 1、fastCGI是使用进程池模式
- 2、主进程负责worker进程的管理
- 3、worker进程负责来自web server的请求的接收与处理
- 4、所有的worker进程都监听同一个端口，现在的linux内核已经完善，不会存在资源抢占。内部会维护一个链表，当有fd的时候，会派发给链表的一个节点
- 5、worker进程分为普通、超标、生病3种状态
- 6、普通worker进程正常执行解析请求
- 7、超标worker进程可能是因为超时或者使用内存过大，这个时候主进程会杀死这个worker进程，并且创建一个新的worker进程。
- 8、生病worker进程，主要是为了防止内存溢出，所以在worker进程执行一定次数的请求后，就主动杀死它，然后创建新的worker进程。

### 2.3、工作原理
- 1、Web Server启动时载入FastCGI进程管理器（IIS ISAPI或Apache Module)
- 2、FastCGI进程管理器自身初始化，启动多个CGI解释器进程(可见多个php-cgi)并等待来自Web Server的连接。
- 3、当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi。
- 4、FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器)(运行在Web Server中)的下一个连接。 在CGI模式中，php-cgi在此便退出了。

### 2.3、特点
- 1、稳定：进程池模式，就算子进程有问题也能马上杀死并创建一个新子进程
- 2、安全：fastCGI支持分布式，与webserver完全脱离，就算fastCGI down了，也不会影响webserver
- 3、性能：相对于CGI模式，性能大大提升，不需要每次都有创建进程的巨大开销。

## 三、php-cgi
CGI解释器进程、用于解析php代码。

## 四、php-fpm
- PHP-FPM是一个PHP FastCGI管理器，是只用于PHP的
- 可以有效控制内存和进程、可以平滑重载PHP配置
- **在php5.4之前，php-fpm是第三方的，没有被官方收录。php-fpm是管理器，php-cgi是解析器，php-fpm管理着php-cgi子进程。**
- **在php5.4之后，php-fpm被官方收录，php-fpm与php-cgi没有关系了。php-fpm子进程实现了php-cgi功能。所以php-fpm既是管理器，又是解析器。**