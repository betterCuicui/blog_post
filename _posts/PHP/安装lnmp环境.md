---
title: 安装lnmp环境
date: 2016-04-28?23:42:25
tags: [php,linux]
category: [PHP]
---
今天安装lnmp环境，发现有个错误，哎，总结一下吧，以防以后又出错
<!--more-->
- 安装nginx
`yum install nginx`
- 安装mysql
`yum install mysql mysql-server`
- 安装php环境
`yum install php-fpm`
- 配置nginx与php相连
`vi /etc/nginx/conf.d/default.conf`
```
location / {
        root   /usr/share/nginx/html;
        index  index.php index.html index.htm;   # ->这行加上index.php
        # example
        #ModSecurityEnabled on;
        #ModSecurityConfig /etc/nginx/modsecurity.conf;
    }
////////////////
location ~ \.php$ {
        root           /usr/share/nginx/html;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /usr/share/nginx/html$fastcgi_script_name;
# 这行一定要注意一下，这里一定要在$前面写上路径，我因为开始这里没写，
# 所以一直报错，打不开php文件，一直说file not found。后来修改好了就好了
        include        fastcgi_params;
    }
```
- 启动服务
service --status-all #查看所有的服务进程
service nginx start/restart
service mysqld start/restart
service php-fpm start/restart