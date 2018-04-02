---
title: 记录：rhel中yum换centos源
date: 2017-12-24 14:25:07
categories: Linux
tags:
  - Linux
---

+ 删除原来的yum

```shell
rpm -qa | grep yum | xargs rpm -e --nodeps
```

+ 使用`curl`或`wget`从`http://mirrors.163.com/centos/7/os/x86_64/`下载yum及其依赖安装包，包括`yum`、`yum-metadata-parser`、`yum-plugin-fastestmirror`、`python-urlgrabber`并安装。如果出现冲突则使用`rpm -i --force --nodeps`强制安装。有可能需要升级`rpm`，同样下载安装包并强制安装。

+ 从`http://mirrors.163.com`的centos帮助文档中获取对应的repo文件，放入`/etc/yum.d.repo/`,然后执行：

```shell

yum clean all
yum makecache

```
