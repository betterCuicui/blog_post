---
title: hexo在新电脑部署以前环境
date: 2017-06-14 10:41:23
tags: [git,blog,hexo]
category: [OTHERS]
---
 
怎么用电脑部署hexo环境，然后搭建github博客的步骤我就不说了，因为网上一搜一大堆。现在讲讲搭建后，怎么转移你的博客到新电脑。
<!--more-->
## 一、部署npm环境
- 安装node.js环境，直接在软件管家里面安装就行了。
- 把node.js弄到环境变量里面
- npm -v看看是不是安装成功
 
## 二、部署hexo环境
- npm install hexo-cli  -g
看到warning不要慌，没有关系
- npm  install
- hexo -v
查看hexo是不是安装成功
 
## 三、转移博客
git的使用方式这里就不说了。如果你以前没有把你的博客的本地内容上传到github，那你就悲剧，不要继续看了。但是记住是你的博客的本地内容，而不是你博客使用hexo生成的内容，**不是你的name.github.io的那个项目**。
- git clone 你的博客的本地内容、环境
 
## 四、上传博客
- hexo s？
发现浏览器输入也不好用了？这个正常，不影响其他使用
- hexo g
- hexo d
**如果发现你的博客乱码了，不要慌，可能是编码的问题，需要是UTF-8 无 BOM格式编码**
 
## 五、踩过的坑
 
#### 5.1、`Cannot find module './db.json'`
执行hexo s的时候，报这个error。解决方法
- 1、copy别人的 `\node_modules\mime-db\db.json`
- 2、在别的目录下，执行`npm install mime-db`，然后把生成的db.json拷贝过去
 
**我使用的是第二种方法**


#### 5.2、` error deployer not found:github `
在执行hexo d的时候，报错
是因为 hexo 更新到3.0之后，deploy的type 的github需要改成git
所以需要执行
```
npm install hexo-deployer-git --save

```

