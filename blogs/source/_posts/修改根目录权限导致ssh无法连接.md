---
title: 修改根目录权限导致ssh无法连接
date: 2019-07-22 08:54:45
categories: Linux
tags:
- ssh
- 权限
---
## 环境
centos:7.2

---
## 背景
因为配置一个服务需要增加一个新账户，不小心将/root 目录下的文件权限都修改为了755，之后再去使用ssh连接就出现“connection closing...Socket close"错误,检查防护墙配置，22端口是否开启，ssh服务配置文件发现均没有问题，所以问题只能是权限相关问题.
这里只修改了权限并没有改变文件属主，只需要将ssh密钥相关的文件权限修改为之前的权限即可，如下：
``` bash
cd /etc/ssh
chmod 644 ./*
chmod 600 ssh_host_dsa_key
chmod 600 ssh_host_rsa_key
chmod 755 .
service sshd restart
```
修改权限后重启ssh服务，之后执行"service sshd status",看到以下内容表明服务开启成功，再次尝试连接就可以了。
![ssh服务状态](/../static/ssh_result.png)