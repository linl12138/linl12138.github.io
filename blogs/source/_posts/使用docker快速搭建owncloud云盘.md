---
title: 使用docker快速搭建owncloud云盘
date: 2019-08-11 09:48:40
categories:
- Linux
tags:
- docker
- owncloud
---

作为百度云盘的老用户，即使没有会员也以就拥有2TB的空间可以使用，但是即使远方有着金山，但通道却比针眼还要细，再小的骆驼也是无法通过的。既然公有云以后看来是不能再享受以前的用户体验，毕竟下载速度即使kb是无法忍受的，那还不如打造自己的云盘。本着不重复造轮子的原则，直接使用owncloud在centos服务器上搭建私有云盘。

## 环境
```
腾讯云 centos7.2
owncloud
docker
```
搭建LAMP环境来部署owncloud是常规的办法，但是会被各种环境的问题困扰，不如直接使用docker来的直接。使用'docker search owncloud'来找到想要的镜像，可以直接使用'docker run'来创建启动容器：
``` yml
docker run -d -p 80:80 owncloud
```
将容器的80端口映射道宿主机的任意端口即可（:前为宿主机制定端口）
但这样操作的灵活性很差，也没有办法进行更多的配置，也可以使用Dockerfile来创建，实际部署中往往要使用到mysql，涉及到镜像排版的问题就要用到docker-compose了，编写docker-compose.yml文件如下：
``` yml
version: '3.1'

services:

  owncloud:
    image: owncloud
    restart: always
    ports:
      - 9090:80

  mysql:
    image: mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: mariadb
```
创建owncloud文件夹，在其中创建docker-compose.yml文件，使用owncloud+mariadb构建，restart参数表示容器可以自动重启。在文件夹内执行
```
docker-compose up -d
```
up命令用于创建并启动容器，-d参数表示后台运行，详细的命令介绍可以看[docker官方文档](http://www.dockerinfo.net/docker-compose-%e9%a1%b9%e7%9b%ae)
使用docker-compose ps命令查看容器的运行状态，确认运行后即可打开对应[服务器地址](http://ip:9090)访问。