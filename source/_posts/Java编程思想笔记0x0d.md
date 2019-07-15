---
title: Java编程思想笔记0x0d
date: 2019-07-12 19:30:15
categories: Java
tags:
  - Java
  - 笔记
---

# Java I/O 系统（三）

## 标准I/O

- 标准I/O是Unix中的概念，指程序所使用的单一信息流。程序的所有输入都可以来自标准输入，所有输出也都可以发送到标准输出，以及所有的错误信息都可以发送到标准错误。其意义在于，可以容易的吧程序传亮起来，一个程序的标准输出可以成为另一个程序的标准输入。

### 从标准输入中读取

- Java提供了`System.in`、`System.out`和`System.err`。`System.out`和`System.err`已经事先被包装成了`PrintStream`对象，`System.in`是没有被包装过的`InputStream`。

```java
public class Test {
    public static void main(String[] args) throws IOException {
        BufferedReader stdin = new BufferedReader(new InputStreamReader(System.in));
        String s;
        while((s = stdin.readLine()) != null && s.length() != 0)
            System.out.println(s);
    }
}
```

### 标准I/O重定向

- Java的`System`类提供一些简单的静态方法调用，以允许我们队标准输入、输出和错误I/O流进行重定向。

```java
public class Test {
    public static void main(String[] args) throws IOException {
        PrintStream console = System.out;
        BufferedInputStream in = new BufferedInputStream(new FileInputStream("Test.java"));
        PrintStream out = new PrintStream(new BufferedOutputStream(new FileOutputStream("test.out")));
        System.setIn(in);
        System.setOut(out);
        System.setErr(out);
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        String s;
        while((s = br.readLine()) != null)
            System.out.println(s);
        out.close();
        System.setOut(console);
    }
}
```

- 上面代码中将标准输入接在了文件上，而标准输出和标准错误重定向至另一个文件。此外，在开始部分保存了最初`System.out`的引用，在最后部分进行了还原。
- I/O重定向操作的是字节流，而不是字符流，因此重定向时注意使用`InputStream`和`OutputStream`。

## 进程控制

- 可以使用Java执行控制台命令，并获取其标准输出、错误。

```java
public class Test {
    public static void main(String[] args) throws IOException {
        OSExecute.command("ping 114.114.114.114");
    }
}
class OSExecute {
    public static void command(String command) {
        boolean err = false;
        try {
            Process process = new ProcessBuilder(command.split(" ")).start();
            BufferedReader results = new BufferedReader(new InputStreamReader(process.getInputStream()));
            String s;
            while ((s = results.readLine()) != null)
                System.out.println(s);
            BufferedReader errors = new BufferedReader(new InputStreamReader(process.getErrorStream()));
            while((s = errors.readLine()) != null) {
                System.err.println(s);
                err = true;
            }
        } catch (Exception e) {
            if(!command.startsWith("CMD /C")) 
                command("CMD /C" + command);
            else 
                throw new RuntimeException(e);
        }
        if (err) {
            System.err.println("Error executing " + command);
        }
    }
}
```

