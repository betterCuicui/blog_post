---
title: redis过期数据存储方式以及删除方式
date: 2017-07-04 19:32:48
tags: [redis]
category: [REDIS]


---

当你通过expire或者pexpire命令，给某个键设置了过期时间，那么它在服务器是怎么存储的呢？到达过期时间后，又是怎么删除的呢？
<!--more-->

## 一、存储方式
比如：
```
redis> EXPIRE book 5
(integer) 1
```
首先我们知道，redis维护了一个存储了所有的设置的key->value的字典。但是其实不止一个字典的。
**redis有一个包含过期事件的字典**
每当有设置过期事件的key后，redis会用当前的事件，加上过期的时间段，得到过期的标准时间，存储在expires字典中。
![](http://1e-gallery.redisbook.com/_images/graphviz-a98e948b7df5bec0a324d87ba3f12bb965e8bd00.png)
从上图可以看出来，比如你给book设置过期事件，那么expires字典的key也为book，值是当前的时间+5s后的unix time。

## 二、删除方式
如果一个键已经过期了，那么redis的如果删除它呢？redis采用了2种删除方式;
#### 2.1、惰性删除
惰性删除的原理理是：放任键过期不管，但是每次从键空间获取键的时候，如果该键存在，再去expires字典判断这个键是不是超时。如果超时则返回空，并删除该键。过程如下：
![](http://1e-gallery.redisbook.com/_images/graphviz-bea3896f8f78c1121ed10c3963085383f28e69df.png)
- 优点：惰性删除对cpu是友好的。保证在键必须删除的时候才会消耗cpu
- 缺点：惰性删除对内存特别不友好。虽然键过期，但是没有使用则一直存在内存中。

#### 2.2、定期删除
redis架构中的时间事件，每隔一段时间后，在规定的时间内，会主动去检测expires字典中包含的key进行检测，发现过期的则删除。在redis的源码redis.c/activeExpireCycle函数中。
下面分别是这个函数的源码与伪代码：


```
void  activeExpireCycle(int type) {
    // 静态变量，用来累积函数连续执行时的数据
    static  unsigned  int current_db =  0; /* Last DB tested. */
    static  int timelimit_exit =  0; /* Time limit hit in previous call? */
    static  long  long last_fast_cycle =  0; /* When last fast cycle ran. */

    unsigned  int j, iteration =  0;
    // 默认每次处理的数据库数量
    unsigned  int dbs_per_call = REDIS_DBCRON_DBS_PER_CALL;
    // 函数开始的时间
    long  long start =  ustime(), timelimit;

    // 快速模式
    if (type == ACTIVE_EXPIRE_CYCLE_FAST) {
        // 如果上次函数没有触发 timelimit_exit ，那么不执行处理
        if (!timelimit_exit) return;
        // 如果距离上次执行未够一定时间，那么不执行处理
        if (start < last_fast_cycle + ACTIVE_EXPIRE_CYCLE_FAST_DURATION*2) return;
        // 运行到这里，说明执行快速处理，记录当前时间
        last_fast_cycle = start;
    }

    /*
    * 一般情况下，函数只处理 REDIS_DBCRON_DBS_PER_CALL 个数据库，
    * 除非：
    * 当前数据库的数量小于 REDIS_DBCRON_DBS_PER_CALL
    * 如果上次处理遇到了时间上限，那么这次需要对所有数据库进行扫描，
    * 这可以避免过多的过期键占用空间
    */
    if (dbs_per_call > server.dbnum  || timelimit_exit)
    dbs_per_call = server.dbnum;

    // 函数处理的微秒时间上限
    // ACTIVE_EXPIRE_CYCLE_SLOW_TIME_PERC 默认为 25 ，也即是 25 % 的 CPU 时间
    timelimit =  1000000*ACTIVE_EXPIRE_CYCLE_SLOW_TIME_PERC/server.hz/100;
    timelimit_exit =  0;
    if (timelimit <=  0) timelimit =  1;

    // 如果是运行在快速模式之下
    // 那么最多只能运行 FAST_DURATION 微秒
    // 默认值为 1000 （微秒）
    if (type == ACTIVE_EXPIRE_CYCLE_FAST)
    timelimit = ACTIVE_EXPIRE_CYCLE_FAST_DURATION; /* in microseconds. */

    // 遍历数据库
    for (j =  0; j < dbs_per_call; j++) {
        int expired;
        // 指向要处理的数据库
        redisDb *db = server.db+(current_db % server.dbnum);

        // 为 DB 计数器加一，如果进入 do 循环之后因为超时而跳出
        // 那么下次会直接从下个 DB 开始处理
        current_db++;

        do {
            unsigned  long num, slots;
            long  long now, ttl_sum;
            int ttl_samples;

            // 获取数据库中带过期时间的键的数量
            // 如果该数量为 0 ，直接跳过这个数据库
            if ((num =  dictSize(db->expires)) ==  0) {
                db->avg_ttl  =  0;
                break;
            }
            // 获取数据库中键值对的数量
            slots =  dictSlots(db->expires);
            // 当前时间
            now =  mstime();

            // 这个数据库的使用率低于 1% ，扫描起来太费力了（大部分都会 MISS）
            // 跳过，等待字典收缩程序运行
            if (num && slots > DICT_HT_INITIAL_SIZE &&
            (num*100/slots <  1)) break;

            // 已处理过期键计数器
            expired =  0;
            // 键的总 TTL 计数器
            ttl_sum =  0;
            // 总共处理的键计数器
            ttl_samples =  0;

            // 每次最多只能检查 LOOKUPS_PER_LOOP 个键
            if (num > ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP)
            num = ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP;

            // 开始遍历数据库
            while (num--) {
                dictEntry *de;
                long  long ttl;

                // 从 expires 中随机取出一个带过期时间的键
                if ((de =  dictGetRandomKey(db->expires)) ==  NULL) break;
                // 计算 TTL
                ttl =  dictGetSignedIntegerVal(de)-now;
                // 如果键已经过期，那么删除它，并将 expired 计数器增一
                if (activeExpireCycleTryExpire(db,de,now)) expired++;
                if (ttl <  0) ttl =  0;
                // 累积键的 TTL
                ttl_sum += ttl;
                // 累积处理键的个数
                ttl_samples++;
            }

            // 为这个数据库更新平均 TTL 统计数据
            if (ttl_samples) {
                // 计算当前平均值
                long  long avg_ttl = ttl_sum/ttl_samples;
                // 如果这是第一次设置数据库平均 TTL ，那么进行初始化
                if (db->avg_ttl  ==  0) db->avg_ttl  = avg_ttl;
                /* Smooth the value averaging with the previous one. */
                // 取数据库的上次平均 TTL 和今次平均 TTL 的平均值
                db->avg_ttl  = (db->avg_ttl+avg_ttl)/2;
            }

            // 我们不能用太长时间处理过期键，
            // 所以这个函数执行一定时间之后就要返回

            // 更新遍历次数
            iteration++;

            // 每遍历 16 次执行一次
            if ((iteration &  0xf) ==  0  &&  /* check once every 16 iterations. */
            (ustime()-start) > timelimit)
            {
                // 如果遍历次数正好是 16 的倍数
                // 并且遍历的时间超过了 timelimit
                // 那么断开 timelimit_exit
                timelimit_exit =  1;
            }

            // 已经超时了，返回
            if (timelimit_exit) return;

            // 如果已删除的过期键占当前总数据库带过期时间的键数量的 25 %
            // 那么不再遍历
        } while (expired > ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP/4);
    }
}
```
**伪代码是：**
```
# 默认每次检测的数据库数量为16
DEFAULT_DB_NUMBERS = 16
# 默认每次检测的键的数量最大为20
DEFAULT_KEY_NUMBERS = 20
# 全局变量，记录当前检测的进度
current_db = 0
def activeExpireCycle():
    # 初始化要检测的数据库数量
    # 如果服务器的数据库数量小于16，则以服务器的为准
    if server.dbnumbers < DEFAULT_DB_NUMBERS:
        db_numbers = server.dbnumbers
    else
        db_numbers = DEFAULT_DB_NUMBERS
        
    # 遍历每次数据库
    for i in range(db_numbers):
        # 如果current_db的值等于服务器的数量，代表已经遍历全，则重新开始
        if current_db = db_numbers:
            current_db = 0
        
        # 获取当前要处理的数据库
        redisDb = server.db[current_db]
        
        # 将数据库索引+1，指向下一个数据库
        current_db++
        
        do
            # 检测数据库中的键
            for j in range(DEFAULT_KEY_NUMBERS):
                # 如果数据库中没有过期键则跳过这个库
                if redisDb.expires.size() == 0:break
            
                # 随机获取一个带有过期事件的键
                key_with_ttl = redisDb.expires.get_random_key()
            
                # 检测键是不是过期了，如果过期则删除
                if is_expired(key_with_ttl):
                    delete_key(key_with_ttl)
            # 已达到时间上限，则停止处理
            if reach_time_limit(): retrun
        while expired>ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP/4
```
对activeExpireCycle进行总结：
- redis默认1s调用10次，这个是redis的配置中的hz选项。hz默认是10，代表1s调用10次，每100ms调用一次。
- hz不能太大，太大的话，cpu会花大量的时间消耗在判断过期的key上，对cpu不友好。但是如果你的redis过期数据过多，可以适当调大。
- hz不能太小，因为太小的话，一旦过期的key太多可能会过滤不完。
- redis执行定期删除函数，必须在一定时间内，超过该时间就return。事件定义为`timelimit =  1000000*ACTIVE_EXPIRE_CYCLE_SLOW_TIME_PERC/server.hz/100` 可以看出该时间与hz成反比，hz默认10，timelimit就为25ms；hz修改为100，那么timelimit就为2.5ms。
- 抽取20个数据进行判断删除为一个轮训，每经过16个轮训才会去判断一次时间是不是超时。
- 如果一个数据库，使用率低于 1%，则不去进行定期删除操作。
- 如果对一个数据库，这次删除操作，已经删除了25%的过期key，那么就跳过这个库。

## 三、redis主从删除过期key方式
当redis主从模型下，从服务器的删除过期key的动作是由主服务器控制的。
- 1、主服务器在惰性删除、客户端主动删除、定期删除一个key的时候，会向从服务器发送一个del的命令，告诉从服务器需要删除这个key。
![](http://1e-gallery.redisbook.com/_images/graphviz-763e7edad215846a6cf6a99999eeace75f99d5de.png)

- 2、从服务器在执行客户端读取key的时候，如果该key已经过期，也不会将该key删除，而是返回一个null
![](http://1e-gallery.redisbook.com/_images/graphviz-9839089a12e44ba12c2237a4c250bfe0e004e15b.png)

- 3、从服务器只有在接收到主服务器的del命令才会将一个key进行删除。

## 四、总结
- 1、expires字典的key指向数据库中的某个key，而值记录了数据库中该key的过期时间，过期时间是一个以毫秒为单位的unix时间戳；
- 2、redis使用惰性删除和定期删除两种策略来删除过期的key；惰性删除只会在碰到过期key才会删除；定期删除则每隔一段时间主动查找并删除过期键；
- 3、当主服务器删除一个过期key后，会向所有的从服务器发送一条del命令，显式的删除过期key；
- 4、从服务器即使发现过期key也不会自作主张删除它，而是等待主服务器发送del命令，这种统一、中心化的过期key删除策略可以保证主从服务器的数据一致性。