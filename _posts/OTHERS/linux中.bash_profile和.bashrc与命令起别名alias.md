---
title: .bash_profile和.bashrc与linux命令起别名alias
date: 2016-09-29 10:47:18
tags: linux
category: OTHERS
---
怎么给linux命中起别名，可以自定义linux命令
<!--more-->
### ~/.bash_profile 与 ~/.bashrc
在刚登录Linux时，首先启动 /etc/profile 文件，然后再启动用户目录下的 ~/.bash_profile、 ~/.bash_login或 ~/.profile文件中的其中一个，执行的顺序为：~/.bash_profile、 ~/.bash_login、 ~/.profile。如果 ~/.bash_profile文件存在的话，一般还会执行 ~/.bashrc文件。因为在 ~/.bash_profile文件中一般会有下面的代码
```
# .bash_profile
 
# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi
 
# User specific environment and startup programs
 
PATH=$PATH:$HOME/bin
 
export PATH
unset USERNAME
```
大家可以看到，里面说如果存在~/.bashrc，那么就执行`.  ~/.bashrc`。
 
然后执行.bashrc的时候，又会去执行/etc/bashrc，可以参考下面的代码。
```
# .bashrc
 
# User specific aliases and functions
 
# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi
 
alias phptest='/home/map/odp_cater/php/bin/php'
#上面这句是后加的。
 
[[ -s "/home/map/.jumbo/etc/bashrc" ]] && source "/home/map/.jumbo/etc/bashrc"
```
 
#### 关于各个文件的作用域，在网上找到了以下说明：
（1）/etc/profile： 此文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行. 并从/etc/profile.d目录的配置文件中搜集shell的设置。
 
（2）/etc/bashrc: 为每一个运行bash shell的用户执行此文件.当bash shell被打开时,该文件被读取。
 
（3）~/.bash_profile: 每个用户都可使用该文件输入专用于自己使用的shell信息,当用户登录时,该文件仅仅执行一次!默认情况下,他设置一些环境变量,执行用户的.bashrc文件。
 
（4）~/.bashrc: 该文件包含专用于你的bash shell的bash信息,当登录时以及每次打开新的shell时,该该文件被读取。
 
（5）~/.bash_logout: 当每次退出系统(退出bash shell)时,执行该文件. 另外,/etc/profile中设定的变量(全局)的可以作用于任何用户,而~/.bashrc等中设定的变量(局部)只能继承/etc /profile中的变量,他们是"父子"关系。
 
（6）~/.bash_profile 是交互式、login 方式进入 bash 运行的~/.bashrc 是交互式 non-login 方式进入 bash 运行的通常二者设置大致相同，所以通常前者会调用后者。
 
**大家从上面的区别可以看出：~/.bash_profile、~/.bashrc和/etc/bashrc都是跟shell有关的，.bashrc就是用来保存一些个人的个性化设置，比如命令的别名、路径等。**
 
### alias命令
当我们需要执行一个php文件的时候，需要`/home/map/odp_cater/php/bin/php a.php`这样是不是 特别麻烦。
alias这个命令是用来设置别名的。大家可以运行alias来看看运行效果。
```
[map@cp01-xiahouyoujie.epc.baidu.com tools]$ alias
alias l.='ls -d .* --color=tty'
alias ll='ls -l --color=tty'
alias ls='ls --color=tty'
alias phptest='/home/map/odp_cater/php/bin/php'
alias vi='vim'
alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
```
从上面的代码中我们看到一条：`alias phptest='/home/map/odp_cater/php/bin/php'`
这个是怎么来的呢？就是因为我们在.bashrc文件中的`alias phptest='/home/map/odp_cater/php/bin/php'`这行代码。可以看到这个命令给php的环境起了一个别名。所以下次再执行php脚本的时候，就可以直接`phptest a.php`了。多么简单。
 
 
#### 重要的是修改了.bashrc文件后，需要执行source ~/.bashrc来重新启动，这样别名就生效了。
 
#### 资料：
- [[/etc/profile、~/.bash_profile等几个文件的执行过程](http://blog.chinaunix.net/uid-14735472-id-3190130.html)](http://blog.chinaunix.net/uid-14735472-id-3190130.html)
- [linux的 .bashrc文件是干什么的?](http://zhidao.baidu.com/link?url=2zKhtvnivy7fNqqZQGrhxasF-JEZ4cgde_YP1OHGps7o19aE3yj8jGnnADfC1mPJ-fs9mA-MhgSaM_zDE5Mrhq)