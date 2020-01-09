---
title: microk8s部署多节点k8s集群
date: 2019-12-31 10:14:42
categories: Kubernetes
tags:
- Kubernetes
- k8s
- docker
- microk8s
---

# microk8s部署多节点Kubernetes集群

为了增加工作量，想上一个k8s。microk8s安装简单还支持多节点，就选它了。

## 准备和安装

snap没有换源一说，只能设置代理。

先设置`systemd editor`的默认编辑器为`vim`

```shell
vim /etc/profile
```

加入以下内容

```shell
export SYSTEMD_EDITOR="/usr/bin/vim"
```

使设置生效

```shell
source /etc/profile
```

配置snapd

```shell
systemctl edit snapd
```

加入以下内容

```ini
[Service]
Environment="http_proxy=http://127.0.0.1:1080"
Environment="https_proxy=http://127.0.0.1:1080"
```

配置生效

```shell
systemctl daemon-reload
systemctl restart snapd
```

安装microk8s

```shell
snap install microk8s --classic
```

查看组件状态

```shell
> microk8s.status
microk8s is running
addons:
cilium: disabled
dashboard: disabled
dns: disabled
fluentd: disabled
gpu: disabled
helm: disabled
ingress: disabled
istio: disabled
jaeger: disabled
juju: disabled
knative: disabled
kubeflow: disabled
linkerd: disabled
metallb: disabled
metrics-server: disabled
prometheus: disabled
rbac: disabled
registry: disabled
storage: disabled
```



## 构建多节点集群

在一台机器上执行

```shell
microk8s.add-node
```

会出现以下内容

```
Join node with: microk8s.join ip-172-31-20-243:25000/DDOkUupkmaBezNnMheTBqFYHLWINGDbf
```

复制`join`指令到其它已经安装了microk8s的机器上执行。

查看集群内节点

```shell
> microk8s.kubectl get no
10.22.254.79       Ready    <none>   27s   v1.15.3
ip-172-31-20-243   Ready    <none>   53s   v1.15.3
```

