---
title: maven的kotlin项目打包以及添加kotlin依赖
date: 2019-12-02 16:09:38
categories: 杂项
tags:
  - kotlin
  - maven
---

# maven的kotlin项目打包以及添加kotlin依赖

准备了一个纯jre的docker镜像，想运行kotlin的jar包，结果找了一圈还是在官网找到了解决办法。

## 不包含kotlin依赖

在`pom.xml`中`build -> plugins`中添加

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>2.6</version>
    <configuration>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <mainClass>${main.class}</mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin>
```



## 包含kotlin依赖

在`pom.xml`中`build -> plugins`中添加

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>2.6</version>
    <executions>
        <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals> <goal>single</goal> </goals>
            <configuration>
                <archive>
                    <manifest>
                        <mainClass>${main.class}</mainClass>
                    </manifest>
                </archive>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
            </configuration>
        </execution>
    </executions>
</plugin>
```

