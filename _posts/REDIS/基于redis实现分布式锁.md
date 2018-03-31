---
title:  基于redis实现分布式锁
date: 2017-09-28 17:26:23
tags: [redis,架构,分布式]
category: [REDIS]
---

因为redis是单进程的，所以在处理请求的时候，是串行化。那么我们可以依赖这个原理来实现分布式锁。
<!--more-->

## 一、setnx
```
$ret = $this->getCon()->setnx($key,1);
if(1 == $ret){
    do something...
    $this->getCon()->del($key);
}
```
- 如果在do something的时候，出现问题导致没有删除锁，那么接下来所有的请求都不能往下执行了。因为锁没有被删，一直被占。

## 二、multi+过期时间
```
$this->getCon()->multi();
$this->getCon()->setnx($key,1);
$this->getCon()->EXPIRE($key,$time);
$this->getCon()->exec();
```
因为sexnx不具备设置过期时间的功能，所以需要借助expire。同时借助multi保证原子性。
- mutil只能保证这两个命令执行的时候，中间不会插入别的命令。但是如果有命令出错，其他命令会继续执行，并不具备事务的回滚功能。
- 所以如果expire命令出问题，那么锁一样会一直被占用。其他命令一样进不来

## 三、set
```
$ret = $this->getCon()->set($key,1,array('nx','ex'=>self::EXP_TIME));
if(1 == $ret){
    do something...
    $this->getCon()->del($key);
}
```
其实sexnx+过期时间的功能，在set命令已经实现了。用这种方式实现的redis分布式锁看起来应该完美了，但是还是有问题的;
- 如果进程A在do something的时候阻塞了，导致进程A的锁超时被释放。
- 进程B获得了锁，执行do something的操作。
- 进程A不在阻塞，然后执行了del的操作，把B的锁给释放了。
- 进程C获得了锁，与进程B并发执行。。。

## 四、通过random实现
```
$ret = $this->getCon()->set($key,$random,array('nx','ex'=>self::EXP_TIME));
if(1 == $ret){
    do something...
    if($random == $this->getCon()->get($key)){
        $this->getCon()->del($key);
    }
}
```
通过random的值，就算redis的key超时后，我们可以通过random的值，也可以不会错误的删除别的进程的锁了。但是还是不完美：
- 进程A获得锁，依次顺序往下执行。
- 进程A执行到`if($random == $reids->get($key))`的操作，发现为true；
- 这个时候，锁刚好超时，被释放了。
- 进程B获得了锁，正常执行。
- 进程A执行了`$this->getCon()->del($key);`的操作，把B的锁给释放了。
- 进程C获得了锁，与进程B并发执行。。。

## 五、Lua脚本保证原子性
```
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```
其实第四种方法为什么有缺点，就是因为在释放锁的时候，分了3步执行
- 获取random
- 对比random
- 删除key

而这三步不能保证原子性，依次执行的中间可能会穿插别的命令。
**那么我们就可以通过Lua脚本来执行，实现释放锁的时候的原子性**
**但是一般，很多proxy并不支持lua脚本，所以当你需要使用lua脚本保证一致性的时候，需要先确定redis集群能否执行lua**