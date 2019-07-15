---
title: Java编程思想笔记0x0f
date: 2019-07-14 16:38:58
categories: Java
tags:
  - Java
  - 笔记
---

# Java I/O 系统（五）

## 压缩

- Java I/O类库中的类支持读写压缩格式的数据流。压缩类库是按字节方式处理的，因此属于`InputStream`和`OutputStream`继承层次结构的一部分。
- 可以使用`InputStreamReader`和`OutputStreamWriter`在两种类型之间进行转换。

### GZIP

- 使用GZIP压缩，直接将输出流封装成`GZIPOutputStream`或`ZipOutputStream`，解压就将输入流封装成`GZIPInputStream`或`ZipInputStream`。

```java
public class GZIPcompress {
    public static void main(String[] args) throws IOException {
        // 检查参数
        if (args.length == 0) {
            System.out.println("no file.");
            System.exit(1);
        }
        // 压缩
        BufferedReader in = new BufferedReader(new FileReader(args[0]));
        BufferedOutputStream out = new BufferedOutputStream(new GZIPOutputStream(new FileOutputStream("test.gz")));
        System.out.println("Writing file");
        int c;
        while ((c = in.read()) != -1) 
            out.write(c);
        in.close();
        out.close();
        // 解压
        System.out.println("Reading file");
        BufferedReader in2 = new BufferedReader(new InputStreamReader(new GZIPInputStream(new FileInputStream("test.gz"))));
        String s;
        while ((s = in2.readLine()) != null)
            System.out.println(s);
    }
}
```

### 用Zip进行多文件压缩、解压

```java
public class ZipCompress {
    public static void main(String[] args) throws IOException {
        // 压缩
        FileOutputStream f = new FileOutputStream("test.zip");
        CheckedOutputStream csum = new CheckedOutputStream(f, new Adler32());
        ZipOutputStream zos = new ZipOutputStream(csum);
        BufferedOutputStream out = new BufferedOutputStream(zos);
        for (String arg: args) {
            System.out.print("Writing file " + arg);
            BufferedReader in = new BufferedReader(new FileReader(arg));
            zos.putNextEntry(new ZipEntry(arg));
            int c;
            while ((c = in.read()) != -1)
                out.write(c);
            in.close();
            out.flush();
        }
    }
}
```




