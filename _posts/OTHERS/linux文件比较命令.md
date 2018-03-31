---
title: linux文件比较命令
date: 2016-03-30 15:27:58
tags: linux
category: OTHERS
---
工作的时候，leader给我两个文件，要我比较一下他们的不同，结果我脑中的第一反应竟然是是用php能不能加载他们，然后把他们的数据用key->value分别读到两个数组中去，现在想想，当初真的比较可爱。
接下来给大家介绍几个linux命令能够特别简单的比较文件的不同
<!--more-->
例子：
```
a.txt：                                 b.txt:
xiahouyoujie                            xiahouyoujie
wangxing                                wangxing
guochunyan                              guochunyan
xiaoxiao                                mazhibin
kris                                    jingjuan
jingjuan                                qiaoqian
qiaoqian                                huangsicong
huangsicong                             liucong
liucong                                 luocong
luocong                                 zhenyunqi
zhenyunqi                               xiahouyuanlu
xiahouyuanlu                            chenyulan
chenyulan
```
任何能够快速的把两个文件的不同瞬间找出来呢？
### 1、uniq命令
uniq [选项] 文件
说明：这个命令读取输入文件，并比较相邻的行。在正常情况下，第二个及以后更多个重复行将被删去
该命令各选项含义如下：
 
– c 显示输出中，在每行行首加上本行在文件中出现的次数。它可取代- u和- d选项。
 
– d 只显示重复行。
 
– u 只显示文件中不重复的各行。
 
– n 前n个字段与每个字段前的空白一起被忽略。一个字段是一个非空格、非制表符的字符串，彼此由制表符和空格隔开(字段从0开始编号)。
 
+n 前n个字符被忽略，之前的字符被跳过(字符从0开始编号)。
 
– f n 与- n相同，这里n是字段数。
 
– s n 与+n相同，这里n是字符数。
 
可能初看这个命令的每个选项中没有我们要的结果，但是我们可以通过别的方式来达到我们的目的啊；
```
cat a.txt | sort | uniq > a1.txt
cat b.txt | sort | uniq > b1.txt
cat a1.txt b1.txt |sort | uniq -d > c.txt    //这个时候c中为a与b 的交集
cat a.txt c.txt | sort | uniq -c | grep '1' > a2.txt
cat b.txt c.txt | sort | uniq -c | grep '1' > b2.txt
//最后将a与b分别与他们的交集做一次集合，显示行数，这样为1行的就是他们的特有的啦。。。
```
 
### 2、Comm命令
 
这个命令就特别简单了，
语法：comm [- 123 ] file1 file2
说明：该命令是对两个已经排好序的文件进行比较。其中file1和file2是已排序的文件。
comm读取这两个文件，然后生成三列输出：仅在file1中出现的行；仅在file2中出现的行；在两个文件中都存在的行。
如果文件名用“- ”，则表示从标准输入读取。
comm -1 不显示只出现在第一个文件的行。
comm -2 不显示只出现在第二个文件的行。
comm -3 不显示同时出现在两个文件的行。
comm file1 file2 显示三列，第一列代表只出现在file1的行，第二列代表只出现在file2的行，第三列代表俩个文件同时出现的行
comm -12 显示两个文件同时出现的行 也就是交集
comm -13 显示只出现在第二个文件的行
comm -23 显示只出现在第一个文件的行
所以如果我们要用这个命令，一样也要通过排序
```
cat a.txt | sort | uniq > a1.txt
cat b.txt | sort | uniq > b1.txt
comm a1.txt b1.txt > c1.txt
cat c1.txt   
            chenyulan
            guochunyan
            huangsicong
            jingjuan
kris
            liucong
            luocong
    mazhibin
            qiaoqian
            wangxing
            xiahouyoujie
            xiahouyuanlu
xiaoxiao
            zhenyunqi
注：第一列为a1.txt特有的 ，第二列为a2.txt特有的 , 第三列为公共的
```
 
### 3、 diff
 
这个命令就比较屌了
不仅说了两个文件的不同，还会告诉你要怎么修改就，两个文件就一样了。而且还不需要你排序，牛。。。
语法：diff [选项] file1 file2
说明：该命令告诉用户，为了使两个文件file1和file2一致，需要修改它们的哪些行。
如果用“- ”表示file1或fiie2，则表示标准输入。如果file1或file2是目录，那么diff将使用该目录中的同名文件进行比较。
diff各选项的含义如下：
- b 忽略行尾的空格，而字符串中的一个或多个空格符都视为相等。
如How are you与How are you被视为相同的字符串。
- c 采用上下文输出格式（提供三行上下文）。
- C n 采用上下文输出格式（提供n行上下文）。
- e 产生一个合法的ed脚本作为输出。
- r 当file1和file2是目录时，递归作用到各文件和目录上。
```
diff a.txt b.txt > diff.txt
cat diff.txt
4,5c4
< xiaoxiao
< kris
---
> mazhibin
‘<‘后面为第一个文件特有的
'>'后面为第二个文件特有的
'4,5c4'是告诉你，把第一个文件的4到5行修改为第二个文件的第4行，这样两个文件就一样啦啦啦。。。
```