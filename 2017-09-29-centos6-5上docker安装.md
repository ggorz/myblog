---
layout: post
title: centos6.5上docker安装
date: 2017-09-29 14:33:06
categories: docker
---


百度百科：docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。
<!-- excerpt -->
<!-- toc -->

cat /proc/version  或
uname -a 或 uname -r
查看Linux内核版本号


### 对于最新版本的docker是否需要升级内核  
To install Docker CE, you need a maintained version of CentOS 7. Archived versions aren’t supported or tested. 只有centos7以上版本才支持docker  
所以我们还是要升级一下内核版本


docker ce的具体安装官方教程  
[https://docs.docker.com/engine/installation/linux/docker-ce/centos/](https://docs.docker.com/engine/installation/linux/docker-ce/centos/)

别人博客教程[https://www.cnblogs.com/saneri/p/6178536.html](https://www.cnblogs.com/saneri/p/6178536.html)

中间有一句执行不成功换成下面这句
rpm -Uvh https://www.elrepo.org/elrepo-release-6-8.el6.elrepo.noarch.rpm


题外： reboot重启命令；
### 升级内核版本
![](https://pic.me33.cn/2017-09-26-153840.jpg)
升级后
![](https://pic.me33.cn/2017-09-26-153908.jpg)
现在可以安装docker了

service docker start  
service docker status  
cat /var/log/docker   // 查看docker日志


### docker ce和ee区别  
 - CE（Community Edition）社区版本
 - EE（Enterprise Edition）企业版本  
 详细区别见: [https://itmuch.com/docker/docker-1/](https://itmuch.com/docker/docker-1/)
