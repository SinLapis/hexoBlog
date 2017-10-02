---
title: VMWare共享文件夹设置
date: 2017-07-19 10:15:45
categories: Linux
tags:
  - Linux
  - VMWare
---
参考：http://www.itread01.com/articles/1476103217.html

讲道理还是在Windows下写代码舒服一些，那么就需要把代码同步到Linux虚拟机上。
使用VMWare的共享文件夹是个好方法。
虚拟机为CentOS7。

<!--more-->

首先给Windows和Linux虚拟机均安装VMWare Tools，下面为Linux下安装VMWare Tools：
```shell
yum install open-vm-tools open-vm-tools-desktop
```
配置服务：
```shell
vi /etc/systemd/system/mnt.hgfs.service
```
内容为：
```
[Unit]

Description=Load VMware shared folders

Requires=vmware-vmblock-fuse.service

After=vmware-vmblock-fuse.service

ConditionPathExists=.host:/

ConditionVirtualization=vmware


[Service]

Type=oneshot

RemainAfterExit=yes

ExecStart=

ExecStart=/usr/bin/vmhgfs-fuse -o allow_other -o auto_unmount .host:/ /mnt/hgfs


[Install]

WantedBy=multi-user.target
```
保存然后启动服务：
```shell
systemctl enable mnt.hgfs.service
```
创建挂载文件夹：
```shell
mkdir -p /mnt/hgfs
```
重启虚拟机应该就能看到共享的文件夹了。
