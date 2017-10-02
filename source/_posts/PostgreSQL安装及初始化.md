---
title: PostgreSQL安装及初始化
date: 2017-08-28 11:11:31
categories: Database
tags:
  - PostgreSQL
  - CentOS
---

参考：https://www.mtyun.com/library/28/how-to-install-postgresql9/
本文安装环境为CentOS 6.4，安装PostgreSQL版本为9.3

<!-- more -->

# 安装ca-certificates

由于官网提供的yum仓库为https，需要安装`ca-certificates`：

```shell
yum install ca-certificates
```

# 安装PostgreSQL 9.3

```shell

yum install http://yum.postgresql.org/9.3/redhat/rhel-6-x86_64/pgdg-centos93-9.3-2.noarch.rpm
yum install postgresql93-server postgresql93-contrib
```

# 配置环境变量

新建数据库物理文件的存放目录：

```shell
mkdir -p /home/postgres/data
```

使用用户`postgres`管理PostgreSQL并配置环境变量：

```shell

su - postgres
cp /etc/skel/.bash* /var/lib/pgsql
```

修改`/var/lib/pgsql/.bashrc`：

```
export PGDATA=/home/postgres/data
export PATH=/usr/pgsql-9.3/bin:$PATH
```

其中`PGDATA`为数据库物理文件的存放目录。
重新加载`.bashrc`：

```shell
source .bashrc
```

# 初始化并启动数据库

```shell

initdb
pg_ctl start
```

# 修改账户密码

在终端输入`psql`进入PostgreSQL交互界面：

```sql
alter user postgres with password 'password';
```
