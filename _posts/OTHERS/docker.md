---
title: docker
date: 2017-06-29 10:26:23
tags: [docker]
category: [OTHERS]


---
docker思想来自于集装箱。在一艘大船上，为了把水果个药品的货物一起运输，那么就需要把水果单独放在集装箱，药瓶单独放在集装箱，这样就不需要专门运输水果或者药品的大船了。而我们现实生产环境中，一台自己的机器，可能需要同时开发多个app，所以会产生很多的问题：
- 不同的应用程序可能会有不同的应用环境，比如.net开发的网站和php开发的网站依赖的软件就不一样
- 你开发软件的时候用的是Ubuntu，但是运维管理的都是centos，运维在把你的软件从开发环境转移到生产环境的时候就会遇到一些Ubuntu转centos的问题
- 在服务器负载方面，如果你单独开一个虚拟机，那么虚拟机会占用空闲内存的，docker部署的话，这些内存就会利用起来

而docker则是把每个app以及所依赖的环境都集中在一个docker。
从此以后我就不用配环境了，也不用装环境了，更不用统一环境了，因为docker都打包好了啊～。所以docker的好处为：
- 1.保证了线上线下环境的一致性
- 2.极大的简化了webapp的部署流程（需要使用某个app模块，直接把该app的docker部署就可以）
- 3.实现了沙盒机制，提高了安全性（app被攻击，不会影响操作系统，直接重新启动一个镜像就可以了）
- 4.实现了模块化，提高了复用性（比如mysql一个docker，这样，别的模块都可以使用mysql的docker）
- 5.实现了虚拟化，提高硬件利用率
<!--more-->

#### docker在centos下的安装
- 移除老版本
```
$ sudo yum remove docker \
                  docker-common \
                  docker-selinux \
                  docker-engine
```
- 安装依赖环境
```
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```
```
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
- 使用edge模式
```
$ sudo yum-config-manager --enable docker-ce-edge
```
```
$ sudo yum-config-manager --enable docker-ce-test
```

- 禁止edge模式（不需要与上面的重复执行）
```
$ sudo yum-config-manager --disable docker-ce-edge
```

- 开始安装
```
$ sudo yum install docker-ce
```
```
systemctl start docker
```
```
docker run hello-world
```

- 结果发现执行hello-world的时候失败，是因为需要rpm包

```
$ sudo yum install /path/to/package.rpm
```
```
$ sudo systemctl start docker
```
```
$ sudo docker run hello-world
```


#### 卸载docker
```
$ sudo yum remove docker-ce
```
```
$ sudo rm -rf /var/lib/docker
```