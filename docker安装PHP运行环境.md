---
title: docker安装PHP运行环境
comments: true
thumbnailImage: https://pic.me33.cn/2018-03-14-052659.png
thumbnailImagePosition: right
date: 2018-03-14
tags:
 - docker
 - linux
categories:
 - linux
keywords:
 - docker
 - php环境
---

转载:[利用docker搭建php7和nginx运行环境全过程](http://www.jb51.net/article/113296.htm)
docker提供了在服务端分布式的部署应用，这样的好处是方便维护和升级。下面这篇文章主要给大家介绍了利用docker搭建php7和nginx运行环境的相关资料，搭建过程中运用的是官方镜像，需要的朋友可以参考借鉴，下面来一起看看吧。

<!-- excerpt -->
<!-- toc -->

## 环境介绍

根目录： /docker

网站根目录：/docker/www

nginx相关目录：/docker/nginx/conf.d

## 准备工作

#### 1.使用docker加速器

```
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://68abbefd.m.daocloud.io
service docker restart
```

执行 service docker restart 报错：

```
[root@joinApp2 ~]# systemctl status docker.service
docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
   Active: failed (Result: start-limit) since Thu 2016-02-25 17:26:11 CST; 16s ago
     Docs: http://docs.docker.com Process: 16384 ExecStart=/usr/bin/docker daemon $OPTIONS $DOCKER_STORAGE_OPTIONS $DOCKER_NETWORK_OPTIONS $ADD_REGISTRY $BLOCK_REGISTRY $INSECURE_REGISTRY (code=exited, status=1/FAILURE)
 Main PID: 16384 (code=exited, status=1/FAILURE)

Feb 25 17:26:10 joinApp2 systemd[1]: Failed to start Docker Application Container Engine.
Feb 25 17:26:10 joinApp2 systemd[1]: Unit docker.service entered failed state.
Feb 25 17:26:10 joinApp2 systemd[1]: docker.service failed.
Feb 25 17:26:11 joinApp2 systemd[1]: docker.service holdoff time over, scheduling restart.
Feb 25 17:26:11 joinApp2 systemd[1]: start request repeated too quickly for docker.service
Feb 25 17:26:11 joinApp2 systemd[1]: Failed to start Docker Application Container Engine.
Feb 25 17:26:11 joinApp2 systemd[1]: Unit docker.service entered failed state.
Feb 25 17:26:11 joinApp2 systemd[1]: docker.service failed.

```

原因：执行了一下curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://*******.m.daocloud.io 语句，然后 在这个文件中的/etc/docker/daemon.json 格式就变了{"registry-mirrors": ["http://****.m.daocloud.io"]，} 最后一个多了一个逗号，我猜测是这个shell语句的原因，我把逗号去掉以后，程序可以正常启动。

#### 2.下载相关镜像

```
docker pull nginx
docker pull php:7.1.0-fpm
```

#### 3.建立相关目录

```
mkdir -p /docker/www
mkdir -p /docker/nginx/conf.d
```

#### 4.编辑default.conf

```
vim /docker/nginx/conf.d/default.conf
```

以下为示例内容

```
server {
  listen  80 default_server;
  server_name _;
  root   /usr/share/nginx/html;

  location / {
   index index.html index.htm index.php;
   autoindex off;
  }
  location ~ \.php(.*)$ {
   root   /var/www/html/;
   fastcgi_pass 172.17.0.2:9000;
   fastcgi_index index.php;
   fastcgi_split_path_info ^((?U).+\.php)(/?.+)$;
   fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
   fastcgi_param PATH_INFO $fastcgi_path_info;
   fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
   include  fastcgi_params;
  }
}
```

## 搭建环境
#### 1.启动php镜像



查看php镜像的ip地址

`docker inspect --format='{{.NetworkSettings.IPAddress}}' myphp`

172.17.0.2

修改default.conf配置文件，使fastcgi_pass的值为 172.17.0.2:9000

`vim /docker/nginx/conf.d/default.conf`

fastcgi_pass 172.17.0.2:9000;

#### 2.启动nginx镜像



#### 3.查看镜像运行状态
docker ps

CONTAINER ID  IMAGE    COMMAND     CREATED    STATUS    PORTS          NAMES
93213e1eac73  nginx    "nginx -g 'daemon off" 3 seconds ago  Up 2 seconds  0.0.0.0:80->80/tcp mynginx
e93281652098  php:7.1.0-fpm  "docker-php-entrypoin" 8 minutes ago  Up 8 minutes 

#### 4.生成php测试文件info.php

`echo "<?php phpinfo();" > /docker/www/info.php`

## nginx虚拟机配置
以配置www.test.com虚拟机为例,项目目录地址为/docker/www/test.com/

```
vim /docker/nginx/conf.d/test.com.conf

# 示例内容如下

server {
  listen  80;
  server_name www.test.com;
  root   /usr/share/nginx/html/test.com/;
  location / {
   index index.html index.htm index.php;
   autoindex off;
  }
  location ~ \.php(.*)$ {
   root   /var/www/html/test.com/;
   fastcgi_pass 172.17.0.2:9000;
   fastcgi_index index.php;
   fastcgi_split_path_info ^((?U).+\.php)(/?.+)$;
   fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
   fastcgi_param PATH_INFO $fastcgi_path_info;
   fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
   include  fastcgi_params;
  }
}


#重启nginx镜像

docker restart mynginx
```

## docker常用命令
1、停止所有正在运行的容器  
docker kill $(docker ps -a -q)  
2、删除所有已停止运行的容器  
docker rm $(docker ps -a -q)  
3、查看容器运行状态  
docker stats  
4、进入容器内进行命令行操作  
docker exec -it content-name-or-id /bin/bash

## 常见问题

```
#############方法一#############
#在宿主主机关闭SELINUX
#临时关闭
setenforce 0
#永久关闭 修改/etc/sysconfig/selinux文件
SELINUX=disabled

#############方法二#############
#以特权方式运行容器
#--privileged参数为true
docker run -it --privileged=true -d nginx
```
