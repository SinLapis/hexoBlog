---
title: 记录：给centos/rhel服务器安装GNOME以及VNC，实现界面远程控制服务器
date: 2017-12-24 15:27:30
categories: Linux
tags:
  - Linux
---

## 服务器部分（CentOS 7）

+ 安装GNOME：

```shell

yum groupinstall -y "GNOME Desktop"

```

+ 安装VNC

```shell

yum install vnc-server vnc*

```

+ 修改`/etc/sysconfig/vncservers`：

```config

VNCSERVERS="1:root"
VNCSERVERARGS[1]="-geometry 1024x768 -alwaysshared -depth 24"

```

+ 添加/修改`/root/.vnc/xstartup`

```config

#!/bin/sh
unset SESSION_MANAGER
[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey vncconfig -iconic &
gnome-session &

```

+ 终端输入`vncpasswd`并根据提示设置密码。

+ 终端输入`vncserver`以启动vncserver。

+ 防火墙开启端口`5901`、`6001`：

```shell

iptables -A INPUT -p tcp --dport 3836 -j ACCEPT
iptables -A INPUT -p tcp --sport 3836 -j ACCEPT
iptables -A INPUT -p tcp --dport 9996 -j ACCEPT
iptables -A INPUT -p tcp --sport 9996 -j ACCEPT

iptables-save

```

## 客户端部分（Windows）

+ 下载`VNC-Viewer`并安装。

+ 建立连接，输入`<ip>:1`，其中`<ip>`为装有VNC的服务器地址，按回车后输入密码即可。
