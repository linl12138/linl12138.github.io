---
title: 记一次yum无法使用的解决
date: 2019-09-08 12:59:04
categories: Linux
tags:
- yum
- DNS
- nsswitch.conf
---

## 环境
腾讯云centos7.2

在一次使用yum安装软件时，yum命令无法执行，错误信息提示是找不到对应主机，开始当然是怀疑yum源出现了问题，所以第一个处理方法是将系统本来的yum源替换掉，这里选择使用网易云的yum源  
首先我们对原来的yum源进行备份，万一不是yum源的问题，也可以切换回来，虽然很多人说腾讯的yum源不是很稳定，但作为大厂，老是崩溃也不太可能。  
``` shell
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
```
然后下载repo：
``` wget http://mirrors.163.com/.help/CentOS7-Base-163.repo
```
生成缓存并更新软件报信息：
``` shell
yum makecache
yum clean all
```
再次尝试yum update,结果和之前一样  
既然不是yum源的问题，那么先把之前的yum源切换为之前的，既然不是yum源的问题，那么会不会是和网络相关的问题，首先使用ping命令尝试去连接一个外部地址，这里以百度为例：
``` shell
ping www.baidu.com
ping请求找不到www.baidu.com主机 请检查该名称，然后重试
```
这里发现了一些问题的头绪，找不到一个正常的主机地址，会不会是DNS服务出现了问题，在验证之前我们改用ip地址再尝试一下，百度的ip地址为202.108.22.5  ``` shell
ping 202.108.22.5
正在 Ping 202.108.22.5 具有 32 字节的数据:
来自 202.108.22.5 的回复: 字节=32 时间=56ms TTL=47
来自 202.108.22.5 的回复: 字节=32 时间=56ms TTL=47
来自 202.108.22.5 的回复: 字节=32 时间=58ms TTL=47
来自 202.108.22.5 的回复: 字节=32 时间=59ms TTL=47

202.108.22.5 的 Ping 统计信息:
    数据包: 已发送 = 4，已接收 = 4，丢失 = 0 (0% 丢失)，
往返行程的估计时间(以毫秒为单位):
    最短 = 56ms，最长 = 59ms，平均 = 57ms
```
我们发现可以ping通ip地址，那么真的可能是DNS的问题了，我们用nslookup去验证一下：
``` shell
nslookup
>www.baidu.com
Server:         183.60.83.19
Address:        183.60.83.19#53

Non-authoritative answer:
www.baidu.com   canonical name = www.a.shifen.com.
Name:   www.a.shifen.com
Address: 180.101.49.12
Name:   www.a.shifen.com
Address: 180.101.49.11
```
奇怪的是域名可以进行解析，我们可以查看/etc/resolve.comf文件，这里存储着DNS服务器信息，一般存在两个，第二个作为备用，打开发现第一个DNS服务器地址正式nslookup命令中使用的DNS服务器地址，所以好像DNS服务也没有问题。  
但是根据之前的ping无法联通域名，说明名称解析相关的服务肯定是有问题的，所以想到了NSS:
根据wikipedia中的说明，NSS(Name Service Switch, 名称服务开关)是类unix操作系统中的一种工具，它为通用配置数据库和名称解析机制提供了各种来源。这些源文件包括本地操作系统文件(例如/etc/passwd、/etc/group和/etc/hosts)、域名系统(DNS)、网络信息服务(NIS)和LDAP。  
我们可以使用nsswitch.conf来对NSS进行配置，联系之前的问题，通过查询可知，ping命令和nslookup处理域名信息是用的并不是同一种方法，使用ping命令处理一个域名的时候，是使用gethostbyname()函数返回对应的主机信息; 而host和nslookup则是直接使用/etc/resolv.conf中的DNS服务器。因此，需要查看/etc/nsswitch.conf中的hosts:数据库是否打开了dns选项。  
那么修改就很容易了，在/etc/nsswitch.conf文件种找到hosts字段，可以发现：  
``` shell
hosts:  files dns
```
要打开DNS选项，只需要在这一行末尾加上wins就可以了，然后发现ping命令可以使用域名了，然后使用'yum update'发现不会报错可以正常执行



