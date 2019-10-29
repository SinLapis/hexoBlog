---
title: React0x00
date: 2019-10-28 16:31:07
categories: React
tags: 
  - 前端
  - React
---

# Hello, world!

创建项目直接使用Webstorm，选择React App。然后执行：

```shell
react-scripts eject
```

清空`src`中的文件，重新建立文件`index.js`，即React入口， 输入：

```jsx
import React from 'react'
import ReactDOM from 'react-dom'

ReactDOM.render(
    <h1>Hello, world!</h1>,
    document.getElementById('root')
);
```

`render`的第一个参数为jsx，可以理解为React的元素，可以作为js变量的值。

```jsx
import React from 'react'
import ReactDOM from 'react-dom'

const ele = <h1>Hello, world!</h1>;
ReactDOM.render(
    ele,
    document.getElementById('root')
);
```

## 创建组件

使用箭头函数。

```jsx
import React from 'react'
import ReactDOM from 'react-dom'

const Ele = (props) => {
    return <h1>Hello, {props.title}!</h1>
};
ReactDOM.render(
    <Ele title="React" />,
    document.querySelector('#root')
);
```



