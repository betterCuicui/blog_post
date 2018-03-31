---
title: linux添加开机启动
date: 2016-09-04 00:03:58
tags: [git,blog,hexo]
category: [OTHERS]
---
如何在linux添加开机启动项呢？
<!--more-->
### 在/etc/rc.local中添加
step1\. 先修改好脚本，使其所有模块都能在任意目录启动时正常执行;
step2\. 再在/etc/rc.local的末尾添加一行以绝对路径启动脚本的行;
如:
**$ vim /etc/rc.local**
```
#!/bin/sh
#
# This script will be executed *after* all the other init scripts.
# You can put your own initialization stuff in here if you don't
# want to do the full Sys V style init stuff.
 
touch /var/lock/subsys/local
. /etc/rc.d/rc.tune
**/opt/pjt_test/test.pl**
```
 
保存并退出;
再重启动测试下，则在其它的程序都启动完成后，将启动脚本;