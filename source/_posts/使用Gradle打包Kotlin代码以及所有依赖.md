---
title: 使用Gradle打包Kotlin代码以及所有依赖
date: 2020-01-14 10:27:12
categories: 杂项
tags:
  - Gradle
  - Kotlin
---

使用插件`Shadow`进行打包。`build.gradle`如下：

```groovy
plugins {
    id 'org.jetbrains.kotlin.jvm' version '1.3.61'
    id 'com.github.johnrengelman.shadow' version '5.2.0'
}

group 'org.example'
version '1.0-SNAPSHOT'

repositories {
    mavenCentral()
    jcenter()
}

dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk8"
    implementation "org.apache.kafka:kafka-clients:2.4.0"
    implementation "org.apache.kafka:kafka-streams:2.4.0"
    implementation "org.apache.logging.log4j:log4j-api:2.13.0"
    implementation "org.apache.logging.log4j:log4j-core:2.13.0"
    implementation "org.slf4j:slf4j-log4j12:1.7.30"
}

compileKotlin {
    kotlinOptions.jvmTarget = "1.8"
}
compileTestKotlin {
    kotlinOptions.jvmTarget = "1.8"
}
jar {
    manifest {
        attributes "Main-Class": "org.example.testK8s.MainKt"
    }
}
```

之后执行下面的指令（或者在IDEA右侧的Gradle面板可以找到）进行打包：

```shell
gradle shadowJar
```



