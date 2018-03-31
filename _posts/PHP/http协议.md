---
title: http协议
date: 2016-04-03 8:20:19
tags: [php]
category: [PHP]
---
身为一个php程序猿，怎么能不了解http呢？所以我觉得我必要总结一下：
但是了解前，温习一下上一篇介绍[输入网址到显示网页过程]比较好。
<!--more-->
### 一、简介
- http协议是超文本传输协议，‘应用层协议’，也是基于tcp/ip的协议；
- http协议基于tcp，为什么不基于udp呢？因为udp首先有不可靠性，也不安全，没有三次握手，服务端发送资源后，都不知道客户端有没有收到。
- http是无状态的协议，每次建立连接后，请求完资源，看http头的Connection，如果keep-alive，则tcp连接会有一天的时间（？这个我也不确定）一般也会填写20s什么的，也会有close，如果是close，服务器就关闭连接，释放资源。因此每次访问都是独立的，跟踪不了用户的，这也是基于性能考虑吧，如果每个用户连接后不都端口，那么服务器的并发是有多大啊。
- http都是一问一答的模式
 
### 二、http请求（包含 请求行、请求头，实体内容）
##### 1、请求头：描述客户端的请求方式、请求资源的名称、http协议的版本号。eg：GET/BOOK/demo.php HTTP/1.1
##### 2、请求头(消息头)包含(客户端请求的服务器的主机名称。客户端的环境信息等)：
- Accept:用户告诉服务端，客户端支持的数据类型(eg:text/html/image/*)。
- Accept-Charset:用于告诉服务器，客户端采用的编码格式（eg：utf-8）
- Accept-Encoding:用于告诉服务器，客户端采用的数据压缩格式
- Accept-Language：客户机语言环境
- Host:客户机通过这个服务器，想访问的主机名
- If-Modified-Since：客户机通过这个头告诉服务器，资源的缓存时间
- Referer：客户机通过这个头告诉服务器，它（客户端）是从哪个资源来访问服务器的（防盗链）
- User-Agent：客户机通过这个头告诉服务器，客户机的软件环境（操作系统，浏览器版本等）
- Cookie：客户机通过这个头，将Coockie信息带给服务器
- Connection：告诉服务器，请求完成后，是否保持连接
- Date：告诉服务器，当前请求的时间
 
##### 3、实体内容（就是指浏览器端通过http协议发送给服务器的实体数据 ）
eg：name=dylan&id=110 （get请求时，通过url传给服务器的值。post请求时，通过表单或者ajax发送给服务器的值）
### 三、http响应（包括响应行、头、内容）
##### 1、响应行
##### 2、响应的头信息
- Location：这个头配合302状态吗，用于告诉客户端找谁
- Server：服务器通过这个头，告诉浏览器服务器的类型
- Content-Encoding：告诉浏览器，服务器的数据压缩格式
- Content-Length：告诉浏览器，回送数据的长度
- Content-Type：告诉浏览器，回送数据的类型
- Last-Modified：告诉浏览器当前资源缓存时间
- Refresh：告诉浏览器，隔多长时间刷新
- Content-Disposition：告诉浏览器以下载的方式打开数据。例如：context.Response.AddHeader("Content-Disposition","attachment:filename=aa.jpg");context.Response.WriteFile("aa.jpg");
- Transfer-Encoding：告诉浏览器，传送数据的编码格式
- ETag：缓存相关的头（可以做到实时更新）
- Expries：告诉浏览器回送的资源缓存多长时间。如果是-1或者0，表示不缓存
- Cache-Control：控制浏览器不要缓存数据
- no-cache Pragma：控制浏览器不要缓存数据
- Connection：响应完成后，是否断开连接。?（close/Keep-Alive）
- Date：告诉浏览器，服务器响应时间
##### 3、响应的内容
比如json，比如html，比如jpg
 
### 三、http请求和响应的共同点
- 每个消息头包含一个头字段名称，然后依次是冒号、空格、值、回车和换行符如： Accept-Encoding: gzip, deflate
 
- 消息头字段名是不区分大小写的，但习惯上讲每个单词的第一个字母大写。 eg：Uuid： xxxx
- 整个消息头部分中的各行消息头可按任何顺序排列。
- 消息头又可分为通用信息头、请求头、响应头、实体头等四类
- 许多请求头字段都允许客户端在值部分指定多个可接受的选项，多个选项之间以逗号分隔。
- 有些头字段可以出现多次，例如，响应消息中可以包含有多个”Warning”头字段。
 
### 四、常用状态码
- 100+ 表示成功接收请求，要求客户端继续提交下一次请求才能完成整个处理过程
- 200+ 表示成功接收请求并已完成整个处理过程
- 300+ 为完成请求，客户需进一步细化请求。例如，请求的资源已经移动一个新地址
- 400+ 客户端的请求有错误
- 500+ 服务器端出现错误
 
## 参考资料
[http://www.blogjava.net/zjusuyong/articles/304788.html](http://www.blogjava.net/zjusuyong/articles/304788.html)