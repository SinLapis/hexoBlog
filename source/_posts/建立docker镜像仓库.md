---
title: 建立docker镜像仓库
date: 2017-10-02 20:11:27
categories: Docker
tags:
  - Docker
  - Linux
---
操作系统为CentOS 7。

# Docker安装

## 安装依赖

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
```

## 添加阿里源

```shell
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

## 安装Docker

```shell
yum install docker-ce
```

## 启动Docker

```shell
systemctl enable docker
systemctl start docker
```

# registry安装

## 安装registry及运行

docker-registry本身也是docker镜像，从官方仓库拉取对应镜像。在仓库服务器上执行：

```shell
docker pull registry:2
```

运行该镜像：

```shell
docker run -d -p 5000:5000 --restart=always --name registry -v /var/data:/var/lib/registry registry:2
```

## 镜像拉取

在仓库服务器上上传本地镜像：

```shell

docker load < centos:1
docker tag centos:1 localhost:5000/centos:1
docker push localhost:5000/centos:1
```

在客户端通过HTTP访问仓库，需要先修改`/etc/docker/daemon.json`：

```json

{
  "insecure-registries":["192.168.59.137:5000"]
}
```

之后输入命令：

```shell

systemctl restart docker
docker pull 192.168.59.137:5000/centos:1
```

## 使用registry API

例如在不知道镜像名称以及版本号时，则需要向registry发起查询。此时可以使用registry提供的API。

```python

import requests
url = 'http://192.168.59.137:5000/v2/'
r = requests.get(url + '_catalog')
print(r.text)
# {"repositories":["busybox"]}
r = requests.get(url + 'busybox/tags/list')
# {"name":"busybox","tags":["1.25.1-musl"]}

```
