---
title: kotlin+nodejs+idea+gradle项目构建
date: 2019-12-11 15:21:40
categories: 杂项
tags:
  - idea
  - gradle
  - nodejs
  - kotlin
---

# kotlin + nodejs + idea + gradle项目构建

kotlin转nodejs的插件只能用gradle，maven虽然也有但是不会用。这玩意我居然搞了一天才看到"Hello, JavaScript!"，人老了。

主要参考：[基于 Node.js 环境的 KotlinJs 工程的完美搭建]( https://www.bennyhuo.com/2019/03/11/kotlin-nodejs/ )。

## 创建项目

现在nodejs已经是IDEA中自带插件了，不再需要手动安装。直接创建项目，选gradle -> kotlin(JavaScript)。

之后需要等IDEA将项目配置完毕。

## 配置build.gradle

```groovy
group 'org.ndp'
version '1'

buildscript {
    ext.kotlin_version = '1.3.60'
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

apply plugin: 'kotlin2js'

buildscript {
    repositories {
        maven {
            url "https://dl.bintray.com/kotlin/kotlin-eap"
        }
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-frontend-plugin:0.0.45"
    }
}

apply plugin: 'org.jetbrains.kotlin.frontend'

repositories {
    mavenCentral()
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib-js:$kotlin_version"
}

compileKotlin2Js {
    kotlinOptions.moduleKind = "commonjs"
    kotlinOptions.sourceMap = true
    kotlinOptions.metaInfo = true
}

kotlinFrontend {
    npm {
        devDependency "karma"     // development dependency
    }
}
```

之后需要静待gradle配置好，时间比较长。

不要在`compileKotlin2Js`中设置输出目录，会导致`Error: Cannot find module 'kotlin'`。

## 编写Kotlin

在`src/main/kotlin/`下新建kt文件：

```kotlin
fun main(args: Array<String>) {
    println("Hello JavaScript!")
}
```

点击`main`旁边的运行即可看到结果。



