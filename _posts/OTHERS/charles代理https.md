---
title: charles代理https
date: 2017-06-28 14:41:23
tags: [tools, charles ]
category: [OTHERS]
---
坑爹的fiddler想代理https，使用各种方法都不行，没办法，直接入charles的怀抱吧，记录一下charles代理https的坑。
<!--more-->
## 一、安装charles
- [安装包](https://share.weiyun.com/609e0ea7900162d4f922ab640bd22d52)
- 如果想使用正版的，直接把`Charles4.0.1\lib\Original\charles.jar`复制出来到lib下就行了
- 如果想使用盗版的，就使用`Charles4.0.1\lib\Crack\charles.jar`

## 二、手机设置IP和端口

## 三、手机安装ssl证书
- 1、手机打开浏览器`chls.pro/ssl`,出现证书安装
- 2、安装证书
- 3、iOS 10.3系统，需要在 _设置→通用→关于本机→证书信任设置_ 里面启用完全信任Charles证书

## 四、charles开启ssl代理
- 打开 proxy->ssl proxy settings->ssl proxying
- 勾选 Enable ssl proxy
- location中添加一个 `*:443`

## 五、charles代理IP
- 打开 tools->maps remote settings
- 勾选 enable map remote
- 添加一条 Map From [Protocol:https] [Host:client.waimai.baidu.com] Map To[Protocol:http] [Host:10.19.160.64] [Port:8159]