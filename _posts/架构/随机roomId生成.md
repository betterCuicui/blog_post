---
title:  随机roomId生成
date: 2017-04-24 11:07:23
tags: [mq,redis,架构]
category: [架构]
---

用户点击创建房间，需要创建一个房间，生成一个随意的roomId，这样别的用户可以根据这个roomId进入房间，roomId是一个6位的，每一位都是类似0-9的数组，比如068390这种，如何生成一个6位数的固定的房roomId呢？
<!--more-->
### 方案一：
```
class  Lib_Common{
    public  function  createRoomId(){
        $arrTemp = array('0','1','2','3','4','5','6','7','8','9');
        shuffle($arrTemp);
        $arrTemp = array_slice($arrTemp,0,Define_Const::ROOM_ID_COUNT);
        $strRoomId = implode('',$arrTemp);
        return $strRoomId;
    }
}
```
- 每次用户创建房间的时候，先用上述的函数生成一个随机6位数。
- 然后去redis执行setNX操作，判断这个roomId是不是已经被使用了。
- 如果没有被使用，则直接使用，告诉用户这个roomId
- 如果已经被使用，则重复上述所有操作
** 
优点：便于理解
缺点，如果用户量大，房间快用完了，这个时候会一直在产生房间号中。
**

### 方案二：
- 首先我们知道，房间号的总量是固定的，一共是：`10*10*10*10*10*10`。所以我们开始可以往redis中，存入所有的房间号，可以用set集合的数据结构。
- 每次用户需要房间号的时候，随机抛出一个。[SPOP key](http://www.redis.net.cn/order/3603.html)，然后用户就能愉快的进行游戏了。
- 当游戏结束后，需要对房间号进行回收。[SADD key member1 [member2]](http://www.redis.net.cn/order/3594.html)这个操作可以抛给mq来执行；
![](/public/image/jiagou/QQ截图20170724110344.png)

**可以很好对比得出，第二种方案是优于第一种的，两种都是为了保证产生的roomId没有被正在使用。第二种只需要一次redis操作。而第一种需要>=1中，而且会随着用户的增加，会操作的次数也会越来越多。**