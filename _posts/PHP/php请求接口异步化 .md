---
title:  php请求接口异步化
date: 2017-08-08 22:53:23
tags: [php,fastcgi_finish_request]
category: [PHP]
---

线上有个接口，耗时特别严重，每次用户操作的时候都会超时。比如需要删除10000个数据，而这些数据只能轮训去删除。每次需要5ms，那么需要50s肯定超时呀。这种情况怎么办呢？
<!--more-->

## 一、修改nginx、php-fpm超时时间
通过修改nginx、php-fpm的超时时间来保证接口能执行完。
#### 1.1、nginx
- nginx：nginx.conf

#### 1.2、php-fpm
- php.ini文件max_execution_time = 100
- set_time_limit(100)
- ini_set('max_execution_time','100');
- 100为100s。如果是0则代表无超时时间

#### 1.3、但是这样的缺点是：
- 1、对于所有的接口，超时时间限制都变大了，不能单独对于某一个接口设置耗时时间，对于线上别的接口性能有要求的场景，不适合。
- 2、如果大量的接口都耗时严重，fast-cgi的子进程都在执行耗时的请求，如果并发量大，则其他请求都处于等待状态，最终可能会造成大量的502.

## 二、mq异步化
对于耗时的请求，我们可以把它放入mq处理，似乎实现了接口的异步化。但是有问题：**目前的mq采用的是主动推的形式**，mq处理请求也是有超时时间的。而我们的超时时间是50s，肯定超过mq的超时时间。怎么办？
这种情况可以同过接口的拆分。把一个耗时严重的请求，一个大动作，拆分成几个小动作。比如：要删除10000个数据，可以每个mq的请求删除1000个，删除10次不就行了。
![](/public/image/jiagou/429d916c-60e4-4706-a310-bfb28e2f3579.png)
当然这种方法的缺点也很明显：
- 1、执行困难，代码比较麻烦
- 2、数据需要加锁处理，很麻烦
- 2、对与实时性要求高的请求也不适合，因为总耗时不会减少，只会增加，可能会造成mq队列任务一直在等待。因为前面的任务执行时间太长

## 三、crontab
使用脚本来操作。
![](/public/image/jiagou/63b586a4-bcd0-49dc-8bbf-720bcf7f6242.png)
#### 3.1、操作步骤
- 1、用户每次请求删除的接口，直接把参数、操作保存在redis的list结构中。
- 2、crontable添加一个cron脚本，每分钟执行一次。
- 3、cron脚本读取redis的list，有值则fork出子进程进行操作。记得要set_time_limit();
- 4、如果cron脚本读取出list中的值，可以先标记一下，表示有脚本正在执行这个操作，如果执行失败还可以把这个任务塞回去再次执行。
- 5、当cron脚本再次发现list有值，先判断有没有脚本正在执行这个操作，有则不管了。

**其实这样是类似自己实现了一个mq。消费者主动拉取任务进行慢慢消化，优点是实现了接口的异步化，对于并发量高的请求也适合。但是这种方法实时性不高，可能一次操作需要1分钟后才能被执行。**

## 四、fastcgi_finish_request
#### 4.1、名词解释：
如果php的执行方式是fastcgi，则有fastcgi_finish_request这个api。此函数冲刷(flush)所有响应的数据给客户端并结束请求。这使得客户端结束连接后，需要大量时间运行的任务能够继续运行。
#### 4.2、函数作用
这个方法可以提高接口的请求速度。如果有些处理可以在页面生成完后再进行,就可以使用这个方法。
比如：
```
<?php
header("Content-Type: application/javascript;charset=utf-8");
echo "正在进行删除操作，请稍后";
fastcgi_finish_request();
for($i = 1-10000){
    删除uid = $i的数据；
}
```
用户通过浏览器请求删除的操作，立马收到'正在进行删除操作，请稍后'的提示，但是数据并没有被删除完，等50s后，才发现数据被删除完毕。**由此说明在调用fastcgi_finish_request后,客户端响应就已经结束,但与此同时服务端脚本却继续运行！**
#### 4.3、注意事项
- 幂等性：如果用户多次请求接口。如果不进行幂等性的保证，后果不堪设想。这种可以通过把用户的操作动作+唯一性（比如`delect:order_id`）setnx到redis，如果返回结果是1，则接口可以执行；是0则返回用户删除操作正在执行。
- 结果同步：因为接口已经异步化了，用户是没有办法能实时知道接口的返回结果，**其实可以用过发送邮件告诉用户接口的执行成功或者失败**

#### 4.4、优缺点：
- 提高了php给客户端的响应时间，可以理解为请求已经异步化。
- 代码简单，方便理解。
- 如果这种请求太多，会造成php-fpm的子进程都在执行这种请求，对后续请求都会造成超时的后果。
- 对于并发量大的请求，拒绝使用这种方式。