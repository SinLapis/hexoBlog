---
title: 使用Java/Kotlin等外部消费者无法消费Kafka消息
date: 2020-01-14 10:07:57
categories: 杂项
tags:
  - Kafka
---

# 问题

有一Kafka集群，在集群内机器上生产消费消息均正常，但是使用外部机器（未部署Kafka）无法消费。

# 解决

在Kafka配置`server.properties`中加入字段：

```properties
port=9092
advertised.host.name=192.168.226.10
```

