---
title: less基础
date: 2019-10-23 10:23:38
categories: less
tags:
  - 前端
  - less
  - css
---

#  less基础

再摸一个less。

## 编译

使用node.js中less编译。

```
npm i -D less
```

```
lessc test.less test.css
```

## 注释

`//`不会出现在css文件中，`/* */`会。

## 变量

支持选择器、属性和属性值。选择器和属性在引用时要使用`{}`，但一般只使用值作为变量。

```less
@mainColor: aliceblue;
@attr: color;
@selector: body;

@{selector} {
  @{attr}: @mainColor;
}
```

less中变量的作用域是块级，并且会延迟加载，即会在块执行完毕后才看变量值。

## 嵌套

less可以实现类似html中的嵌套。

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title><%= htmlWebpackPlugin.options.title %></title>
</head>
<body>
<div id="root">
    outer
    <h1>test word.</h1>
    <h1 class="green">hello, world!</h1>
</div>
</body>
</html>
```

```less
@mainColor: aliceblue;

body {
  color: @mainColor;
  h1 {
    color: coral;
  }
  .green {
    color: aqua;
  }
}
```

设置元素状态时需要修正嵌套级别。

```less
@mainColor: aliceblue;

body {
  color: @mainColor;
  h1 {
    color: coral;
  }
  .green {
    color: aqua;
    &:hover {
      color: #66cccc;
    }
  }
}
```

## 混合

可以理解为less的方法？

可以省去大段重复css代码。

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title><%= htmlWebpackPlugin.options.title %></title>
</head>
<body>
<div id="root">
    outer
    <h1 id="text1">test word.</h1>
    <h1 id="text2">hello, world!</h1>
</div>
</body>
</html>
```

```less
@mainColor: aliceblue;

.textHover {
  color: aqua;
  &:hover {
    color: #66cccc;
  }
}
body {
  color: @mainColor;
  h1 {
    color: coral;
  }
  #text1, #text2 {
    .textHover;
  }
}
```

混合的css会输出到编译后的css文件里，会有重复，可以使用如下的方式来避免。

```less
@mainColor: aliceblue;

.textHover() {
  color: aqua;
  &:hover {
    color: #66cccc;
  }
}
body {
  color: @mainColor;
  h1 {
    color: coral;
  }
  #text1, #text2 {
    .textHover;
  }
}
```

混合可以带有参数。

```less
@mainColor: aliceblue;

.textHover(@origin, @hover) {
  color: @origin;
  &:hover {
    color: @hover;
  }
}
body {
  color: @mainColor;
  h1 {
    color: coral;
  }
  #text1 {
    .textHover(aqua, #44cccc);
  }
  #text2 {
    .textHover(pink, #dda0ba)
  }
}
```

混合中参数可以有默认值。

```less
@mainColor: aliceblue;

.textHover(@origin: aqua, @hover: #44cccc) {
  color: @origin;
  &:hover {
    color: @hover;
  }
}

body {
  color: @mainColor;

  h1 {
    color: coral;
  }

  #text1 {
    .textHover;
  }

  #text2 {
    .textHover(pink, #dda0ba)
  }
}
```

参数有默认值时可以指定部分参数。



