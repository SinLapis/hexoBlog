---
title: Java8实战笔记0x0b
date: 2019-08-25 11:06:04
categories: Scala
tags:
  - Java
  - Scala
  - 笔记
---

# 面向对象和函数式编程的混合：Java 8和Scala的比较

## *Java 与 Scala的混合编程*

[参考](https://www.cnblogs.com/yjmyzz/p/4694219.html)。使用Maven生成项目，`pom.xml`文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.bigmt</groupId>
    <artifactId>mjns</artifactId>
    <version>1.0-SNAPSHOT</version>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>11</java.version>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>${maven.compiler.source}</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-library</artifactId>
            <version>2.13.0</version>
        </dependency>
        <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-compiler</artifactId>
            <version>2.13.0</version>
        </dependency>
        <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-reflect</artifactId>
            <version>2.13.0</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
                <configuration>
                    <release>11</release>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.scala-tools</groupId>
                <artifactId>maven-scala-plugin</artifactId>
                <version>2.15.2</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>testCompile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

![项目结构](/images/2019-08-25_111145.png)

```java
// Main.java
package org.bigmt.mjns;

public class Main {
    public static void main(String[] args) {
        ScalaMain scalaMain = new ScalaMain();
        scalaMain.entry();
    }
}
```

```scala
// ScalaMain.scala
package org.bigmt.mjns

class ScalaMain {
  def entry(): Unit = {
    var n: Int = 2
    while (n <= 6) {
      println(s"Hello ${n} bottles of beer")
      n += 1
    }
  }
}
```

## Scala简介
### 你好，啤酒

```scala
// Scala
2 to 6 foreach {n => println(s"Hello ${n} bottles of beer")}
```

```java
// Java
IntStream.rangeClosed(2, 6)
    .forEach(n -> System.out.println("Hello " + n + " bottles of beer"));
```

### 基础数据结构 

#### 创建集合

```scala
// Scala
val authorsToAge = Map("Raoul" -> 23, "Mario" -> 40, "Alan" ->53)
val authors = List("Raoul", "Mario", "Alan")
val numbers = Set(1, 1, 2, 3, 5, 8)
```

#### 不可变

- Scala中不可变的集合是持久化的，更新一个Scala集合会生成一个新的集合，这个新的集合和之前版本的集合共享大部分内容，最终的结果是数据尽可能地实现了持久化。

```scala
// Scala
val numbers = Set(2, 5, 3)
val newNumbers = numbers + 8
println(numbers)
println(newNumbers)
/* Output:
Set(2, 5, 3)
Set(2, 5, 3, 8)
*/
```

#### 使用集合

```scala
// Scala
val fileLine = List("test", "test for scala", "no vac", "yeeeeeeees man")
val linesLongUpper = fileLine.par.filter(_.length() > 10).map(_.toUpperCase())
println(linesLongUpper)
/* Output
List(TEST FOR SCALA, YEEEEEEEES MAN)
*/
```

