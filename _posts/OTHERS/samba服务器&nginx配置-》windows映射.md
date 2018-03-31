---
title: samba服务器&nginx配置->windows映射
date: 2016-04-29 17:09:03
tags: [php,linux,nginx]
category: OTHERS
---
昨天安装好了lnmp环境，今天该弄编译器了，哈哈，下载了一个phpstrom，但是刚刚开始就碰壁，phpstrom可以直接通过ftp连接linux编程，但是我一直连接不上，notpad++也下载插件来用过ftp来实习远程编程。所以放弃，想到公司可以直接映射到本地的硬盘，嘿嘿，发现用的是samba服务器，所以就。。。
<!--more-->
## 安装samba服务器
`yum install samba samba-client samba-swat`
- samba-common-3.5.10-125.el6.x86_64               //主要提供samba服务器的设置文件与设置文件语法检验程序testparm
- samba-client-3.5.10-125.el6.x86_64                    //客户端软件，主要提供linux主机作为客户端时，所需要的工具指令集
- samba-swat-3.5.10-125.el6.x86_64                    //基于https协议的samba服务器web配置界面
- samba-3.5.10-125.el6.x86_64                            //服务器端软件，主要提供samba服务器的守护程序，共享文档，日志的轮替，开机默认选项
 
## 配置samba服务器
`vi /etc/samba/smb.conf `
- security = user/share
user是通过用户名和密码来访问
share则不需要
- 其他的我没动，动了一下，发现不好使
 
## 添加samba用户
```
useradd [username]
smbpasswd -a [username]
```
然后启动samba服务器
`service samba restart`
 
## 注意：
- 服务器没有启动，发现是防火墙的事，直接service iptables stop；
1、使用Samba服务器需要防火墙开放以下端口
    UDP 137
    UDP 138
    TCP 139
    TCP 445
vi /etc/sysconfig/iptables   #配置防火墙端口
需要写在-A INPUT -m state --state------下面，格式需要对其
```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 139 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 445 -j ACCEPT
-A INPUT -m state --state NEW -m udp -p udp --dport 137 -j ACCEPT
-A INPUT -m state --state NEW -m udp -p udp --dport 138 -j ACCEPT
/etc/rc.d/init.d/iptables restart     #重启防火墙，使规则生效
```
- 要是这个时候，你发现samba还是没有启动，那可能是selinux的事情了；
    selinux是linux的内部防火墙；
    直接在命令行敲：
    `setenfore 0`
    ```
    vi /etc/selinux/config
    将SELINUX=enforcing改为SELINUX=disabled为开机重启后不再执行setenfore节约光阴
    ```
- 这个时候发现，你添加什么用户，就能在windows映射出/home/[username]这里的文件,别的文件我们是看不到的，或者没有权限去看到.
- 但是因为我们需要的是/usr/share/nginx/html这里的文件，那怎么办呢？我开始想通过samba配置去实现，后来放弃了，可能因为这个是root下的，所以还是通过去修改nginx访问的根目录比较好
 
## 修改nginx访问的跟目录
```
将其中的
location / {
  root  html;
  index  index.php index.html index.htm;
  }
 
改为
  location / {
  root   /home/www;
  index index.php index.html index.htm;
  }
 
然后再将
location ~ \.php$ {
  root   html;
  fastcgi_pass  127.0.0.1:9000;
  fastcgi_index  index.php;
  fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
  include  fastcgi_params;
  }
 
改为
location ~ \.php$ {
  root   /home/www;
  fastcgi_pass  127.0.0.1:9000;
  fastcgi_index  index.php;
  fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
  include  fastcgi_params;
  }
  ```
这样nginx访问的根目录就修改好了，结果发现访问的时候是403错误，原来是权限不够，果断chmod 777