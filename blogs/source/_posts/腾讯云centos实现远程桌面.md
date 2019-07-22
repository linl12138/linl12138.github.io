---
title: 腾讯云centos实现远程桌面
date: 2019-07-22 08:55:27
categories:
- Linux
tags:
- 腾讯云
- centos
- 远程桌面
- VNC
- KDE
---

腾讯云提供的云服务器选择安装centos或者ubuntu系统默认是最小化安装,没有提供桌面安装，作为一名非Linux服务器运维人员，没有桌面
的系统用起来还是很多不方便，而且因为个人主力机是Windows，需要能够实现远程桌面连接，在权衡实用与性能等因素，最终选择的环境如下：

---
```
系统：centos 7.2;
桌面选择：KDE;
远程桌面：VNC,xrdp；
```
---
Linux桌面的发行版有非常多种，但真正好用的并不很多，要么陈旧，要么有些丑陋，这里没有使用一般默认的Gnome桌面，主要原因就是
以为它有点丑，毕竟和mac或者Windows的桌面进行比较的话，Linux还是洗洗睡吧。

## 安装桌面
首先通过腾讯云提供的命令行登录方式进入系统界面，先看看有那些可以安装的桌面。
``` bash
yum grouplist
```
在Available Environment Group 中我们可以找到可以安装的桌面类型，可以选择Mate，Gnome护着Server with GUI,但使用下来发现KDE桌面更胜一筹，可以使用yum来安装桌面，但在安装桌面之前我们要安装一个桌面与系统之间的接口程序"X Windows system",具体可参考[X Windows system](https://en.wikipedia.org/wiki/X_Window_System)。
安装前先更新一下yum源：
``` bash
yum update
```
然后安装X Windows system
``` bash
yum groupinstall "X Windows system"
```
此处需要注意安装程序名需要加引号，否则会出错。
安装KDE桌面：
``` bash
yum groupinstall "KDE Plasma Workspaces" #KDE桌面名
```
桌面需要手动启动：
``` bash
startx
```
这个过程可能会很长，耐心等待即可。执行完成后可以先使用腾讯云提供的VNC登录的方式登录看一下是否桌面已经启动。桌面安装完成后很多会先重启一下，嗯，这是个好习惯，但此时你会再次看到系统只有 命令行界面，是桌面没安装上吗，继续使用yum grouplist命令查看，发现在已经安装项中已经有了KDE桌面，或者是没有启动，再次执行startx，或许有用，那要是再次重启可又不行了。系统在没有安装桌面 的时候默认启动的是命令行模式，那当只是安装了桌面后系统并不知需要启动桌面，所以需要修改系统的默认启动模式。先来检查一下:
``` bash 
systemctl get-default
```
得到的结果是“multi-user.target”，这显然不是界面启动，我们修改这个启动项：
``` bash
systemctl set-default graphical.target
```
再次重启就可以了。

---
## 安装VNC
接下俩需要实现从windows远程，首先安装VNC相关程序：
``` bash
yum -y install vnc-server
yum -y install tigervnc-server
```
安装完成后我们需要修改一下配置文件，位置为/lib/systemd/system目录下，可以发现有一个vncserver@.service的文件，先复制一份：
``` bash 
cp /lib/systemd/system/vncserver@.service /lib/systemd/system/vncserver@:1.service
```
修改复制后的文件：将Execstart行中的<USER>修改为自己的账户名，修改PIDFile中的文件路径为用户目录路径：
``` 
ExecStart=/usr/sbin/runuser -l root -c "/usr/bin/vncserver %i"
PIDFile=/root/.vnc/%H%i.pid
```
我使用的均为root账户，可以根据实际情况修改。也可以使用-geometry参数修改分辨率，这里直接使用默认。VNC默认使用的是5901端口，如果有防火墙，将5901端口打开：  
``` bash
firewall-cmd --permanent --zone=public --dport=5901/tcp
iptables -A INPUT -p tcp -dport=22 -j ACCEPT
```
启动vncserver，直接键入vncserver,此时需要给vnc账户添加一个密码，然后运行：
``` bash
systemctl daemon-reload
systemctl enable vncserver@:1.service #使用新添加的服务
reboot
systemctl start vncserver@:1.service
```
此时就可以使用Windows进行远程桌面，可以下载使用[vnc viewer](https://www.realvnc.com/en/connect/download/viewer/windows/),也可以使用Windows自带的mstsc服务进行远程。
首次登录可能会失败，可以先关闭防火墙再次连接即可。