---
title: 使用K3s搭建Kubernetes集群
date: 2020-01-09 11:07:46
categories: Kubernetes
tags:
- Kubernetes
- k8s
- docker
- k3s
---

# 使用K3s搭建Kubernetes集群

主要参考[你的第一次轻量级K8S体验 —— 记一次Rancher 2.2 + K3S集成部署过程](https://blog.ilemonrain.com/docker/rancher-with-k3s.html)

需要至少2台机器。

当前各软件版本为：

| 软件名  | 版本          |
| ------- | ------------- |
| Docker  | 18.09.9       |
| K3s     | v1.17.0+k3s.1 |
| rancher | v2.3.3        |



## Docker安装

直接参考[官方文档](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

> 安装依赖使`apt`可以使用HTTPS的仓库
>
> ```shell
> apt install -y \
>     apt-transport-https \
>     ca-certificates \
>     curl \
>     gnupg-agent \
>     software-properties-common
> ```
>
> 添加Docker官方的GPG密钥
>
> ```shell
> curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
> ```
>
> 添加Docker稳定版仓库
>
> ```shell
> add-apt-repository \
>    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
>    $(lsb_release -cs) \
>    stable"
> ```
>
> 更新`apt`索引
>
> ```shell
> apt update
> ```
>
> 安装Docker
>
> ```shell
> apt-get install -y docker-ce docker-ce-cli containerd.io
> ```

配置registry

```shell
docker pull registry:2
docker run -d -p 5000:5000 --restart=always --name registry \
    -v /var/data:/var/lib/registry registry:2
```

修改Docker配置```/etc/docker/daemon.json```

```json
{
	  "insecure-registries" : ["192.168.226.11:5000"]
}
```

## rancher安装

选择一台安装rancher server，这台不安装k3s，好像会有冲突。

```shell
docker run -d -v /data/docker/rancher-server/var/lib/rancher/:/var/lib/rancher/ --restart=unless-stopped --name rancher-server -p 80:80 -p 443:443 rancher/rancher:stable
```

之后访问`http://192.168.226.10/`（改成自己的ip）进行初始化部署。设定密码和地址后，可选切换语言。

选择`添加集群 -> 导入`，输入集群名，然后点击`创建`。保存最后一条指令，记得在`kubectl`前添加`k3s`。

## k3s安装

到[k3s - Releases](https://github.com/rancher/k3s/releases)下载`k3s`文件（约50MB的那个），上传至`/usr/local/bin/`，并执行下面指令

```shell
chmod +x /usr/local/bin/k3s
```

下载`pause`镜像

```shell
docker pull kubernetes/pause:latest
docker tag kubernetes/pause:latest k8s.gcr.io/pause
```

执行官方安装脚本

```shell
curl -sfL https://get.k3s.io | sh -
```

配置容器引擎使用Docker

```shell
vim /etc/systemd/system/multi-user.target.wants/k3s.service
```

在文件中`ExecStart`字段最后一个`\`后添加一行，并填入`--docker`，类似下面

```ini
ExecStart=/usr/local/bin/k3s \
    server \
    --docker
```

保存配置并重启`k3s`

```shell
systemctl daemon-reload
systemctl restart k3s
```

执行之前保存的指令，注意不要直接使用下面的样例，主机地址和`yaml`文件名要使用刚才生成的。

```shell
curl --insecure -sfL https://192.168.226.11/v3/import/nb4hcqpzsvggwhcsfgpj5vjss8s2wsqbhv82d72s68hx8cf6gfzhsj.yaml | k3s kubectl apply -f -
```

稍等一会界面就会出现节点状态。

## 容器执行测试

参考[Kubernetes官方文档](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/)

先准备一个镜像

```shell
docker pull busybox
docker tag busybox:latest 192.168.226.11:5000/busybox:latest
docker push 192.168.226.11:5000/busybox:latest
```

假设现在我们从私有仓库拉取一个镜像并执行。

创建`job.yaml`文件

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: 192.168.226.11:5000/busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

创建`Job`

```shell
k3s kubectl create -f job.yaml
```

查看`Job`状态

```shell
kubectl get cronjob hello
```

查看`Job`创建的`Pod`

```shell
kubectl get jobs
```

选择上面的一个已完成的`pod`的id，例如`hello-1578545280`。获取这个`pod`的输出

```shell
pods=$(kubectl get pods --selector=job-name=hello-1578545280 --output=jsonpath={.items[*].metadata.name})
kubectl logs $pods
```

## 添加节点

新节点同样先安装Docker。安装k3s按如下步骤。

查看主节点的`node-token`

```shell
cat /var/lib/rancher/k3s/server/node-token
```

在新节点上安装并启动为普通节点，注意ip地址和token改成自己的。

```shell
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.226.11:6443 K3S_TOKEN=K105933dce21eca704fb3913c26976e0c13c36878fc0c846a0780915c12fccdd78e::server:1a8e2a73e1247868ccb5b3ce0b3cbc7e sh -
```

同样需要配置容器引擎使用Docker

```shell
vim /etc/systemd/system/multi-user.target.wants/k3s-agent.service 
```

修改方式和主节点一样，然后重启`k3s-agent`，在`rancher`界面即可看到新加入的节点。