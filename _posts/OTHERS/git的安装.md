---
title: git的安装与基本使用
date: 2016-03-30 23:10:21
tags: [git,linux]
category: [OTHERS]
---
工作的时候，git的使用是必不可少的，然后我总结了一下git的安装与使用
<!--more-->
## 一、设置git的username和email
```
$git config --global user.name 'cuicui'   //global全局？？
$git config --global user.email 'youjiexiahou@163.com'
```
## 二、生成ssh密匙，这个东西主要是让github能够识别你的电脑，这样你就不用每次输入帐号密码了
#### 1、查看是不是有了ssh密钥：
 
* windows：直接查看用户->.ssh
* linux：cd ~/.ssh
 
#### 2、生成密钥
 
```
$ssh-keygen -t rsa -C 'youjiexiahou@163.com'
连按三个回车，密码为空
```
 
最后得到了两个文件:id_rsa和id_rsa.pub
#### 3、在github添加你的ssh公钥
 
 
## 三、开始使用github
#### 1、获取源码
 
```
$git init
$git clone git@github.com.........git  [filename，不添加则克隆到当前目录]
```
#### 2、设置git命令简写
哈哈，偷懒是人类必须的啊
```
//global 是全局变量，这样的话所有的git命令都能简写了
//而过加了global，这样的话，这个配置就会加入git的安装目录下的config文件了
//如果不加global，那么git的简写就只能针对当前仓库了
git config --global alias.st status
git config --global alias.ci commit
git config --global alias.br branch
git config --global alias.unstage 'reset HEAD'
git config --global alias.last 'log -1'
//这样git last就能显示最近一次提交了
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```
 
 
#### 3、git提交代码
```
git pull origin master
结果提示 ：
fatal: 'origin' does not appear to be a git repository
fatal: Could not read from remote repository.
解决方法是：
git remote add origin git@········.git
 
 
//一些基本的操作
git fetch   //相当于是从远程获取最新版本到本地，不会自动merge
git br             //查询当前所有的分支
git br [brname]   //新建分支,一般新建的时候，需要先切换到master，因为新建的分支会跟你当前分支代码一样
git co [brname] //切换分支
git status //查询当前所有修改的文件
git diff [filename] //查询具体修改了什么内容，精确到行
git co [filename] //把修改的文件切换回上一个版本add或者ci版本，自动删除你这次的修改部分
git add [filename] //把修改的文件提交到临时栈
git rm [filename] //从git中删除指定文件
git ci -m 'name' //提交临时栈中的代码到master库
git push origin br //上线
git merge origin/dev //将分支dev与当前分支进行合并
git br -D [brname] //删除本地库develop
 
 
//如果有冲突的时候
git pull origin master //把master的代码拉取下来
 
 
//stash 可用来暂存当前正在进行的工作,比如想pull 最新代码，又不想加新commit，或者另外一种情况，为了fix一个紧急的bug,  先stash, 使返回到自己上一个commit, 改完bug之后再stash pop, 继续原来的工作。
git stash push //将文件给push到一个临时空间中
git stash list //显示临时空间的
git stash pop stash@{0} //将文件从临时空间pop下来,最上面的那个
 
 
//git 回滚到某个版本
git log //查询日志，里面每个版本会有唯一值
git reset --hard <commit ID号> //ci号在log中会有
```
## 四、git注意事项
 
#### 1、大git不包含小git
意思就是，如果一个文件夹a中有git，然后a文件夹包含b文件夹，但是b文件夹中也有git，这个时候，如果a问价git push，他不会把b文件push上去。所以说大文件git不包含小文件git
 
#### 2、gitignore
当你上传的时候，有一些文件是不需要上传的，但是你又想`git add .`，那怎么办？
直接在工程文件下创建一个文件.gitignore,然后里面直接写不需要上传的文件，eg:
```
*/a.html
temp/*
c.html
```