---
title: kotlin+puppeteer写爬虫
date: 2019-12-11 17:00:58
categories: 杂项
tags:
  - kotlin
  - 爬虫
  - puppeteer
---

# kotlin + puppeteer写爬虫

环境搞的很郁闷，这个代码倒是简单解决了，多亏了一位日本老哥。

主要参考：[バックグラウンドで使うpuppeteer with Kotlin]( https://qiita.com/numa08/items/214c6c9d06d5094add3a )

## 环境准备

build.gradle中添加：

```groovy
dependencies {
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core-js:1.1.1'
}
```



## 让kotlin使用async/await

接口有变动，参考中代码部分失效。

```kotlin
import kotlin.coroutines.*
import kotlin.js.Promise

suspend fun <T> Promise<T>.await(): T = suspendCoroutine { cont ->
    then({ cont.resume(it) }, { cont.resumeWithException(it) })
}

fun <T> async(x: suspend () -> T): Promise<T> {
    return Promise { resolve, reject ->
        x.startCoroutine(object : Continuation<T> {
            override val context = EmptyCoroutineContext

            override fun resumeWith(result: Result<T>) {
                if (result.isSuccess)
                    resolve(result.getOrNull()!!)
                else
                    reject(result.exceptionOrNull()!!)
            }
        })
    }
}
```

## 封装puppeteer接口

```kotlin
import kotlin.js.Promise

@Suppress("FunctionName")
@JsModule("puppeteer")
external object Puppeteer {

    class Page {

        fun goto(url: String, options: dynamic): Promise<dynamic>

        fun waitFor(element: String, options: dynamic): Promise<dynamic>

        fun waitFor(num: Int): Promise<dynamic>

        fun content(): Promise<dynamic>

        fun click(selector: dynamic): Promise<dynamic>

        fun close(): Promise<dynamic>

        fun evaluate(pageFunction: Function<dynamic>): Promise<dynamic>

    }

    class Browser {

        fun newPage(): Promise<Page>

        fun close(): Promise<dynamic>

        fun wsEndpoint(): String

    }

    fun launch(options: dynamic): Promise<Browser>
}
```

## 爬虫代码

```kotlin
fun main() {
    async {
        val browser = Puppeteer.launch(object {}.also { it: dynamic ->
            it.devtools = true
            it.args = arrayOf("--no-sandbox", "--disable-setuid-sandbox")
            it.headless = true
        }).await()
        try {
            val page = browser.newPage().await()
            page.goto("http://www.baidu.com", object {}.also { it: dynamic -> it.timeout = 10 * 1000 }).await()
            page.waitFor(1000).await()
            val content = page.content().await()
            println(content.toString())
        } finally {
            browser.close().await()
        }
    }
}
```

注意`it.headless = true`为开启Chrome的Headless模式，需要显示界面调试置为`false`即可。