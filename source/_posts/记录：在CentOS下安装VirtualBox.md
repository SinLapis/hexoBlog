---
title: 记录：在CentOS下安装VirtualBox
date: 2017-12-25 13:10:54
categories: Linux
tags:
  - Linux
---

+ 安装`kernel-devel`，注意和系统内核版本对应。查看系统内核版本：

```shell

uname -r

```

+ 安装其他依赖：

```shell

yum install -y SDL gcc make perl

```

+ 到官网上下载CentOS对应的rpm安装包，下载并安装即可。
