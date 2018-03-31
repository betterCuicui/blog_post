---
title: insert与唯一索引问题
date: 2017-06-14 12:41:23
tags: [mysql,php]
category: [MYSQL]
---
 
一个需求，需要使用insert，会命中唯一索引，然后就研究了一下
<!--more-->
## 需求：创建白名单
```
CREATE TABLE `antispam_white_list` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增id',
  `phone` bigint(20) unsigned NOT NULL COMMENT '用户手机号',
  `white_type` tinyint(3) unsigned NOT NULL DEFAULT '0' COMMENT '白名单类型',
  `create_time` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '创建时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_phone_white` (`phone`,`white_type`),
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
- phone和white_type字段组成了唯一索引
- 主键是id
- phone是索引
- create_time是索引
 
### 问题：给用户添加白名单的时候，会命中唯一索引怎么办？
 
#### 解决一：判断唯一索引存不存在
通过sql中WHERE not exists来判断，这个太low
 
 
#### 解决二：ON DUPLICATE KEY UPDATE
如果命中索引需要更新期中某条数据则可以使用这个方法：
```
insert into antispam_white_list (phone,white_type,create_time,editor) VALUES (18745710310,2,1496820854,'xiahouyoujie')ON DUPLICATE KEY UPDATE create_time=VALUES(create_time)
```
 
#### 解决三：REPLACE INTO
如果命中索引，先执行del删除原来的数据，然后再执行insert。
```
REPLACE into antispam_white_list (phone,white_type,create_time,editor) VALUES (18745710310,2,1496820854,'xiahouyoujie')ON DUPLICATE KEY UPDATE create_time=VALUES(create_time)
```
**所以假设这条数据命中唯一索引，那么返回结果会是2，因为先执行了del，再insert。2次操作都成功返回2。而且符合事务原子性，数据一致性，如果insert失败，数据会回滚**
 
#### 解决四：INSERT IGNORE into
如果批量插入，命中索引，则不更新这条数据，则使用这种解决方法
```
INSERT IGNORE INTO antispam_white_list (phone,white_type,create_time,editor) VALUES (18745710310,2,1496820854,'xiahouyoujie')ON DUPLICATE KEY UPDATE create_time=VALUES(create_time)
```
假设这条数据存在，则返回0，因为并没有执行任何操作。