---
title: jOOQ逆向生成Java代码
date: 2019-08-15 11:32:24
categories: Java
tags:
  - Java
  - jOOQ
---

# jOOQ逆向生成Java代码

## 引入

要操作一下GeoIP库，打算接触一下ORM。又因为最近在看Stream，想找一种类似流水线的Java处理数据库相关的框架。jOOQ符合了我的需求。

[官网文档连接](<https://www.jooq.org/doc/3.11/manual-single-page/#jooq-in-7-steps-step3>)

## 关于xml

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<configuration xmlns="http://www.jooq.org/xsd/jooq-codegen-3.11.0.xsd">
  <!-- Configure the database connection here -->
  <jdbc>
    <driver>com.mysql.jdbc.Driver</driver>
    <url>jdbc:mysql://localhost:3306/library</url>
    <user>root</user>
    <password></password>
  </jdbc>

  <generator>
    <!-- The default code generator. You can override this one, to generate your own code style.
         Supported generators:
         - org.jooq.codegen.JavaGenerator
         - org.jooq.codegen.ScalaGenerator
         Defaults to org.jooq.codegen.JavaGenerator -->
    <name>org.jooq.codegen.JavaGenerator</name>

    <database>
      <!-- The database type. The format here is:
           org.jooq.meta.[database].[database]Database -->
      <name>org.jooq.meta.mysql.MySQLDatabase</name>

      <!-- The database schema (or in the absence of schema support, in your RDBMS this
           can be the owner, user, database name) to be generated -->
      <inputSchema>library</inputSchema>

      <!-- All elements that are generated from your schema
           (A Java regular expression. Use the pipe to separate several expressions)
           Watch out for case-sensitivity. Depending on your database, this might be important! -->
      <includes>.*</includes>

      <!-- All elements that are excluded from your schema
           (A Java regular expression. Use the pipe to separate several expressions).
           Excludes match before includes, i.e. excludes have a higher priority -->
      <excludes></excludes>
    </database>

    <target>
      <!-- The destination package of your generated classes (within the destination directory) -->
      <packageName>test.generated</packageName>

      <!-- The destination directory of your generated classes. Using Maven directory layout here -->
      <directory>C:/workspace/MySQLTest/src/main/java</directory>
    </target>
  </generator>
</configuration>
```



上面直接贴了官网样例。可能需要修改的有：

- 数据库连接`jdbc`
- 要生成Java代码对应的数据库名`inputSchema`
- 目标路径`directory`和目标包名`packageName`

## 生成步骤

首先按官网文档所说，下载`jooq-3.11.11.jar`、`jooq-codegen-3.11.11.jar`、`jooq-meta-3.11.11.jar`以及`mysql-connector-java-5.1.36.jar`四个jar包到某个临时目录（例如`D:\temp`），将上面的xml文件也放进该目录（命名为`x.xml`）。

```
java -classpath jooq-3.11.11.jar;jooq-meta-3.11.11.jar;jooq-codegen-3.11.11.jar;mysql-connector-java-5.1.36.jar;. org.jooq.codegen.GenerationTool x.xml
```

如果参考了官网的命令，注意去掉谜之换行以及MySQL驱动jar包名称后面的`-bin`。