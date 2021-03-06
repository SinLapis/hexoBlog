---
title: MongoDB笔记0x02
date: 2017-10-23 10:28:36
categories: Database
tags:
  - MongoDB
---

## 2.7 MongoDB shell 的使用

由于使用了V8引擎，MongoDB shell支持运行JavaScript代码，并且如果在启动MongoDB客户端时在`mongo`后面添加js文件，即可在shell可交互之前运行指定的js代码。如果有些js文件需要频繁运行，可以把这部分JavaScript代码写入一个名为`.mongorc.js`的文件，并放在用户主目录下；这时MongoDB shell运行时就会先执行`.mongorc.js`。

`mongorc.js`中，可以：

（1）创建需要的全局变量；
（2）简化过长的名字，为其起一个别名；
（3）重写内置函数；
（4）移除比较危险的函数，例如`dropDatabase`以及`deleteIndexes`,让它们不能执行，防止误操作。这条是比较常用的。

shell中的提示是可以定制的，例如在查询过程中显示时间以及显示当前使用的Da名。

如果在shell中希望修改之前已经输入的行，可以在`.mongorc.js`中添加一行：

```js

EDITOR = "/usr/bin/vi"

```

如此，需要修改某个变量时可以输入`edit post`即可修改`post`变量。

shell与集合命名：如果集合名中包含JavaScript的保留字或者不允许的字符，将不能使用`db.collectionName`的形式来访问集合，需要使用函数`db.getCollection()`来代替，也可以使用类似于`db['collectionName']`来访问。

## 2.8 服务器端脚本

在服务器上执行脚本务必注意安全性问题。如果配置不当，很容易遭受注入攻击。例如：

```js

func = "function() { print('Hello, '" + name + "); }"

```

此时用户如果传入的`name`为：

```js

name = "'); db.dropDatabase(); print('"

```

这样和原脚本中拼接起来是合法的JavaScript代码时，就造成了注入攻击，导致数据库被删除。防范注入攻击主要是确保不要将用户输入的内容直接传递给mongod，例如应对上述的注入攻击可以使用作用域，以下为python的例子：

```python

func = pymongo.code.Code("function() { print('Hello, ' + username + '!'); }", {"username": name})

```

如此再输入之前的内容，则输出结果会变为：

```

Hello, '); db.dropDatabase(); print('!

```

# 3 创建、更新和删除文档

## 3.1 插入并保存文档

批量插入：除了使用`insert`插入单条数据，还可以使用`batchInsert`进行批量插入，它接受一个文档数组。MongoDB接受的最长消息为48MB，如果超过48MB，多数的驱动程序会将插入请求拆分成多个。如果在插入的过程中某个文档插入错误，则这个文档之前的文档均插入成功，之后的文档均插入失败。可以设置`continueOnError`参数，跳过插入有误的文档，继续进行插入。所有的驱动程序均支持该参数，但shell不支持。

插入校验：插入文档时，会进行基本的数据校验，例如检查有没有`_id`字段，没有则会自动生成一个；会检查文档的大小，单个文档的大小不能超过16M，以防止不良的设计模式，保持性能的一致；会检查有误非utf-8字符存在等等。但尽管如此，向MongoDB中插入非法数据是十分容易的，所以应当只允许信任源连接数据库是必要的。

## 3.2 删除文档

使用`remove()`进行文档删除时，会删除集合中的所有文档，但不会删除集合本身，以及集合的元信息。`remove()`也可以接受一个条件参数，来删除指定的文档。删除时永久性的，不可撤销，也不可恢复。

使用`drop()`的删除速度要远远超过`remove()`，但是如此之后集合也会被删除，也不能指定条件。
