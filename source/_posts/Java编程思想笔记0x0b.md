---
title: Java编程思想笔记0x0b
date: 2019-07-09 20:14:59
categories: Java
tags:
  - Java
  - 笔记
---

# Java I/O 系统

## File

- `File`类既代表一个特定文件的名称，又能代表一个目录下的一组文件的名称。

  ```java
  public class Main {
      public static void main(String[] args) {
          File path = new File(".");
          String[] list;
          if(args.length == 0)
              list = path.list();
          else 
              list = path.list(new DirFilter(args[0]));
          Arrays.sort(list, String.CASE_INSENSITIVE_ORDER);
          for(String dirItem: list)
              System.out.println(dirItem);
      }
  }
  
  class DirFilter implements FilenameFilter {
      private Pattern pattern;
      public DirFilter(String regex) {
          pattern = Pattern.compile(regex);
      }
      @Override
      public boolean accept(File dir, String name) {
          return pattern.matcher(name).matches();
      }
  }
  ```

  - 上面代码中，`File#list()`可以接受一个`FilenameFilter`对象，用于过滤文件。此处使用了回调，重载了`accept()`方法并让`list()`调用。

  ## 输入和输出

  ### InputStream类型

  - `InputStream`的作用是用来表示那些从不同数据源产生的输入的类，包括：字节数组、`String`对象、文件、管道、由其他种类的流组成的序列等，每一种数据源都有相应的`InputStream`子类。

| 类                        | 功能                                                         | 构造器参数                                                   | 如何使用                                                     |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `ByteArrayInputStream`    | 允许将内存的缓冲区当做`InputStream`使用                      | 缓冲区，字节将从中取出                                       | 作为一种数据源：将其与`FilterInputStream`对象相连以提供有用接口 |
| `StringBufferInputStream` | 将`String`转换成`InputStream`使用                            | 字符串，底层是实现使用`StringBuffer`                         | 作为一种数据源：将其与`FilterInputStream`对象相连以提供有用接口 |
| `FileInputStream`         | 用于从文件中读取信息                                         | 字符串，表示文件名、文件或`FileDescriptor`对象               | 作为一种数据源：将其与`FilterInputStream`对象相连以提供有用接口 |
| `PipedInputStream`        | 产生用于写入相关`PipedOutputStream`的数据，实现管道化的概念  | `PipedOutputStream`                                          | 作为一种数据源：将其与`FilterInputStream`对象相连以提供有用接口 |
| `SequenceInputStream`     | 将两个或多个`InputStream`对象转换成单一`InputStream`         | 两个`InputSteam`对象或者一个容纳`InputStream`对象的容器`Enumeration` | 作为多线程中数据源：将其与`FilterInputStream`对象相连以提供有用接口 |
| `FilterInputStream`       | 抽象类，作为装饰器的接口。其中，装饰器为其它的`InputStream`类提供有用的功能 | -                                                            | -                                                            |

​    