---
title: Java编程思想笔记0x13
date: 2019-07-24 14:25:30
categories: Java
tags:
  - Java
  - 笔记
---

# 注解

- 注解也被称为元数据，为在代码中添加信息提供了一种形式化的方法，以便在后面使用这些数据。
- 有三种内置的注解，分别是：
  1. `@Override`，表示当前的方法定义将覆盖超类中的方法。如果注解的方法没有对应到超类中的方法，编译器会发出错误提示。
  2. `@Deprecated`，如果在其它代码中使用了注解为`@Deprecated`的元素，那么编译器会发出警告信息。
  3. `@SuppressWarnings`，关闭不当的编译器警告信息。

## 基本语法

### 定义注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@interface Test {}
```

- 除了`@`以外，`@Test`的定义很像一个空的接口。
- 没有元素的注解称为标记注解。

### 元注解

| 元注解        | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| `@Target`     | 表示该注解可以用于什么地方，可能的参数包括：`CONSTRUCTOR`（构造器声明），`FIELD`（域声明），`LOCAL_VARIABLE`（局部变量声明），`METHOD`（方法声明），`PACKAGE`（包声明），`PARAMETER`（参数声明），`TYPE`（类、接口、注解、枚举） |
| `@Retention`  | 表示需要再什么级别保存该注解信息。可选的参数包括：`SOURCE`（注解将被编译器丢弃），`CLASS`（注解在class文件中可用，但会被VM丢弃），`RUNTIME`（VM在运行期也会保留注解，可以通过反射读取注解信息） |
| `@Documented` | 将此注解包含在Javadoc中                                      |
| `@Inherited`  | 允许子类继承父类中的注解                                     |

## 编写注解处理器

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@interface UseCase {
    public int id();

    public String desc() default "No description";
}

class PasswordUtils {
    @UseCase(id = 47, desc = "Need one or more numeric")
    public boolean validatePassword(String password) {
        return (password.matches("\\w*\\d\\w*"));
    }
    @UseCase(id = 48)
    public String encryptPassword(String password) {
        return new StringBuilder(password).reverse().toString();
    }
    @UseCase(id = 49, desc = "Same with the previous")
    public boolean chekcForNewPassword(List<String> prevPasswords, String password) {
        return !prevPasswords.contains(password);
    }
}

public class Main {
    public static void trackUseCases(List<Integer> useCases, Class<?> cl) {
        for (Method m: cl.getDeclaredMethods()) {
            UseCase uc = m.getAnnotation(UseCase.class);
            if (uc != null) {
                System.out.println(uc.id() + " " + uc.desc());
                useCases.remove((Integer)uc.id());
            }
        }
        for (int i: useCases) {
            System.out.println(i + "Not found");
        }
    }
    public static void main(String[] args) throws Exception {
        List<Integer> useCases = new ArrayList<>();
        Collections.addAll(useCases, 47, 48, 49, 50);
        trackUseCases(useCases, PasswordUtils.class);
    }
}
```

- 上面代码使用了反射来取得注解信息。其中，`getAnnoation()`方法返回指定类型的注解对象。如果被注解方法上没有该类型的注解，则返回`null`值。如果注解中没有指定注解字段值，则返回定义注解字段时的默认值。

### 注解元素

- 注解元素可用的类型：

  - 所有基本类型
  - `Class`
  - `enum`
  - `String`
  - `Annotation`

  如果使用了其它类型则会报错。另外，注解可以作为元素的类型，这意味着注解可以嵌套。

### 默认值限制

- 元素必须有默认值，或者在使用注解时提供元素的值。
- 对于非基本类型的元素，无论是在源代码中，还是在注解接口中定义默认值，都不能以`null`作为其值。