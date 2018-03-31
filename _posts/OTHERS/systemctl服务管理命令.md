---
title: systemctl服务管理命令
date: 2017-06-28 16:38:58
tags: [linux,systemctl]
category: [OTHERS]
---


重新安装了centos，启动服务的时候，学习了systemctl命令。 这个是系统服务管理器命令，它实际上将 service 和 chkconfig 这两个命令组合到一起，特别强大，也很方便。
<!--more-->

## 一、常用命令
- **1、启动服务**
```
systemctl start smb.service //启动是samba服务
systemctl start firewalldfirewalld.service //启动防火墙服务
```
-  **2、查看服务状态 systemctl status smb.service**
```
➜  ~ systemctl status smb.service
● smb.service - Samba SMB Daemon
   Loaded: loaded (/usr/lib/systemd/system/smb.service; disabled; vendor preset: disabled)
   Active: active (running) since 三 2017-06-28 20:25:56 CST; 1min 36s ago
 Main PID: 2372 (smbd)
   Status: "smbd: ready to serve connections..."
   CGroup: /system.slice/smb.service
           ├─2372 /usr/sbin/smbd
           ├─2373 /usr/sbin/smbd
           ├─2374 /usr/sbin/smbd
           └─2375 /usr/sbin/smbd
```
- **3、停止服务**
```
systemctl stop smb.service
```
- **4、设置开机启动**
```
systemctl enable smb.service
```
- **5、取消开机启动**
```
systemctl disable smb.service
```
- **6、重启服务**
```
systemctl resatrt smb.service
```
- **7、查看所有已启动服务**
```
systemctl list-units --type=service
```

## 二、查看服务状态
- 1、查看所有可用单元
```
systemctl list-unit-files
```

- 2、查看所有正在运行单元
```
systemctl list-units
```

- 3、查看所有服务
```
systemctl list-unit-files --type=service
```

- 4、查看所有正在运行的服务
```
systemctl list-units --type=service
```

## 三、重启、关机命令
- 1、重启
```
systemctl reboot
```
- 2、停止
```
systemctl halt
```
- 3、挂起
```
systemctl suspend
```
- 4、休眠
```
systemctl hibernate
```
- 5、混合休眠
```
systemctl hybrid-sleep
```

