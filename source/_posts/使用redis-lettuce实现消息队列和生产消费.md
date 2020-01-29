---
title: 使用redis+lettuce实现消息队列和生产消费
date: 2020-01-29 09:05:52
categories: redis
tags:
  - redis
  - kotlin
  - lettuce
---

# 准备

## 安装redis

使用Docker部署redis

```shell
docker run --name redis-mq -d -p 6379:6379 redis redis-server --appendonly yes
```

## 创建stream和group

安装redis-cli

```shell
apt install -y redis-cli
```

进入redis-cli

```shell
redis-cli -h localhost -p 6379
```

创建空的stream和对应的group

```
xgroup create testTask testGroup $ mkstream
```

# 代码实现

## 配置依赖

使用lettuce作为redis连接库

```kotlin
dependencies {
    implementation(kotlin("stdlib-jdk8"))
    implementation("io.lettuce:lettuce-core:5.2.1.RELEASE")
}
```

## 生产者

```kotlin
fun consume() {
    val client = RedisClient.create("redis://192.168.75.120:6379/0")
    val connection = client.connect()
    val commands = connection.sync()
    val consumer = io.lettuce.core.Consumer.from("testGroup", "testTask")
    val content = commands.xreadgroup(
        consumer, XReadArgs.StreamOffset.lastConsumed("testTask")
    )
    println(content.size)
    for (c in content) {
        for (k in c.body.keys) {
            println("$k: ${c.body[k]}")
        }
    }
}
```

## 消费者

```kotlin
fun main() {
    val client = RedisClient.create("redis://192.168.75.120:6379/0")
    val connection = client.connect()
    val commands = connection.sync()
    val body = HashMap<String, String>()
    for (i in 1..10) {
        body[i.toString()] = "test"
    }
    commands.xadd("testTask", body)
}
```

