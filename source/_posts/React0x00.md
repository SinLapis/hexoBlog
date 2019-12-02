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

# 创建组件

## 箭头函数

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

## 类

```jsx
import React from 'react'
import ReactDOM from 'react-dom'

class Ele extends React.Component {
    render() {
        return (
            <div>
                <h1>class app</h1>
                <p>{this.props.title}</p>
            </div>
        )
    }
}

ReactDOM.render(
    <Ele title="arg"/>,
    document.querySelector('#root')
);
```

### 组件嵌套

```jsx
import React from 'react'
import ReactDOM from 'react-dom'

const Title = () => <h1>app title</h1>;

class Ele extends React.Component {
    render() {
        return (
            <div>
                <Title />
                <p>{this.props.title}</p>
            </div>
        )
    }
}

ReactDOM.render(
    <Ele title="test arg"/>,
    document.querySelector('#root')
);
```

# 组件样式

## 使用js

```jsx
const Title = () => <h1 style={{color: '#0099ff'}}>app title</h1>;
```

## 使用class

```css
.blue-title {
    color: #0099ff;
}
```

```jsx
import './index.css'

const Title = () => <h1 className="blue-title">app title</h1>;
```

## 使用第三方库classnames

安装。

```shell
npm i -D classnames
```

```css
.blue-title {
    color: #0099ff;
}
.red-title {
    color: #ff6666;
}
.bg {
    background-color: antiquewhite;
}
```

```jsx
import './index.css'
import classNames from 'classnames'

const Title = () => <h1 className={classNames('bg', {'blue-title': true, 'red-title': false})}>app title</h1>;
```





