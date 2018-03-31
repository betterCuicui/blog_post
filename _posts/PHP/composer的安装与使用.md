---
title: composer的安装与使用
date: 2017-06-14 11:45:23
tags: [php,composer]
category: [PHP]
---
 
发现一个神器，cpmposer，简单介绍一下怎么使用
<!--more-->
### 一、安装
- 执行在线安装
```
curl -sS https://getcomposer.org/installer | php
```
然后会在你的本地获得一个`composer.phar`的文件。
- 把composer.phar移动到bin下
```
mv composer.phar /usr/local/bin/composer
composer -V
```
接下来就可以直接使用composer了
 
### 二、使用
- 创建一个composer.json文件，写入相应的包名和版本号
```
{
  "require": {
      "monolog/monolog": "1.0.*"
  }
}
```
- 执行 `composer install`，就进入自动安装，安装完成后会生成一个`composer.lock`文件，里面是特定的版本号名，需要这个文件和`composer.json`一起提交到版本管理里去。
 
- 更新依赖
`composer update`
如果只想更新部分依赖
`composer update monolog/monolog`
 
- 自动加载
先引用加载类
`require 'vendor/autoload.php';`
然后在php中这样使用：
```
      $log = new Monolog\Logger('name');
      $log->pushHandler(new Monolog\Handler\StreamHandler('app.log', Monolog\Logger::WARNING));
 
      $log->addWarning('Foo');
```