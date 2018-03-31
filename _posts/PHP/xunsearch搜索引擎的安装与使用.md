---
title: xunsearch搜索引擎的安装与使用
date: 2016-09-22 20:07:19
tags: [php]
category: [PHP]
---
 
首先说一下[xunsearch的官网](http://www.xunsearch.com/)
毕设的爬虫项目需要用到中文搜索引擎了，所以本来想弄sphinx的，但是sphinx是不支持中文的，不然需要加coreseek的，太麻烦了。而我刚好又在网上看到了xunsearch，就抱着试试的态度安装玩了一下，结果大大出乎我的意料。。。
<!--more-->
### 安装
安装教程官网上有，特别简单，这里我就不解释了。
安装后启动
`/usr/local/xunsearch/bin/xs-ctl.sh restart`
当然可以把这行代码添加进`/etc/rc.local`这样就能开机启动了
### 给mysql建立索引
- 首先在  xunsearch安装目录/sdk/php/app里面添加   school
_news.ini
 
```
project.name = school_news//这个需要跟文件名一样
project.default_charset = utf-8
server.index = 8383
server.search = 8384
 
[id]
type = id
 
[tittle]
type = title
 
[contents]
type = body
 
[url]
type = string
 
[public_time]
type = date
```
具体的那些type或者别的什么东西的意思可以看[网上介绍](http://www.xunsearch.com/doc/php/guide/ini.guide)
- 导入mysql的数据到xunsearch里面
```
php /usr/local/xunsearch/sdk/php/util/Indexer.php --rebuild --source=mysql://yourname:yourpasswd@127.0.0.1/yourdatabase --sql="select id,url,title,contents,public_time from school_spider" --project=school_news
```
这样建立索引的事情就完成了.
### 使用
官网讲的很详细，这里我就把自己的例子弄上去
 
```
require '/usr/local/xunsearch/sdk/php/lib/XS.php';
//导入api
$xs = new XS('school_news');
$search = $xs->search;//创建搜索对象
 
$docs = $search->setFuzzy()->setQuery($search_contents)->setLimit(50)->search();
//开始搜索，setFuzzy()为模糊查询
 
$res = array();
foreach ($docs as $doc)
{
  $temp = array();
  $temp['tittle'] = $search->highlight($doc->tittle); // 高亮处理 tittle 字段
  $temp['contents'] = $search->highlight($doc->contents); // 高亮处理 contents 字段
  $temp['url'] = $doc->url;
  $temp['public_time'] = $doc->public_time
  $res[] = $temp;
}
```
这个东西很强大，以后有时间好好研究研究.