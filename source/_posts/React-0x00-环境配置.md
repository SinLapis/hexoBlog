---
title: React-0x00-环境配置
date: 2019-10-22 09:34:40
categories: React
tags:
  - 前端
  - React
  - Webpack
---

# React开发环境配置

Kotlin看不动了，摸鱼学学前端。

## Webpack配置

先安装node.js和npm，此处略。

### 初始化

初始化package描述文件，在项目路径中生成`package.json`文件。

```
npm init
```

或者使用`npm init --yes`省去中间配置步骤。

### 安装Webpack

```
npm i webpack webpack-cli -D
```

> --save-dev(-D)参数意思是把模块版本信息保存到devDependencies（开发环境依赖）中，即package.json的devDependencies字段中

### 配置build启动脚本

在`package.json`中，对`scripts`添加`build`字段：

```json
{
  "scripts": {
    "build": "webpack --mode production",
    "test": "echo \"Error: no test specified\" && exit 1"
  }
}
```

此外在项目目录中新建文件夹`src`作为源码目录，在`src`中新建文件`index.js`作为程序入口，内容随便写点。

此时可以测试能否build成功：

```
npm run build
```

项目目录中会出现一个`dist`文件夹，里面有一个`main.js`，即转换生成的js文件。

### 使用webpack.config.js

可以使用`webpack.config.js`来自定义`build`的入口和出口。该文件可以使用`node.js`的模块。

```js
const path = require("path");

module.exports = {
    entry: './src/home.js',
    output: {
        path: path.resolve(__dirname, "dist"),
        filename: 'output.js'
    }
}
```

`require`是`node.js`引入模块的方法。此处引入的`path`是用于处理操作系统中文件路径的模块。`path.resolve()`方法将路径转换为绝对路径。`__dirname`为当前执行文件所在目录的完整目录名。

如果需要有多个入口和多个出口，可以如下配置：

```js
const path = require("path");

module.exports = {
    entry: {
        main: './src/home.js',
        about: './src/about.js'
    },
    output: {
        path: path.resolve(__dirname, "dist"),
        filename: '[name].js'
    }
};
```

`output`可以省略，生成的文件名使用`entry`中对应的`key`值。`[name]`是由`webpack`自动填充的字段。还可以使用`[hash]`、`[chunkHash]`来生成hash值。

`webpack.config.js`的位置也是可配置的，需要修改`package.json - scripts - build`字段

```json
{ 
  "scripts": {
    "build": "webpack --mode production --config scripts/webpack.config.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  }
}
```

此时`webpack`的当前路径变为`./scripts/`，因此还需要修改`webpack.config.js`。

```js
const path = require("path");
const dist = path.join(__dirname, '..', 'dist');

module.exports = {
    entry: {
        main: './src/home.js',
        about: './src/about.js'
    },
    output: {
        path: dist,
        filename: '[name].js'
    }
};
```

或者

```js
const path = require("path");

module.exports = {
    entry: {
        main: './src/home.js',
        about: './src/about.js'
    },
    output: {
        path: path.resolve(process.cwd(), "dist"),
        filename: '[name].js'
    }
};
```

`process.cwd()`为获得当前执行node命令时候的文件夹目录名。

### 自动生成html

如果生成js使用了hash值作为文件名，那么对应的html也需要自动生成。安装插件`html-webpack-plugin`。

```
npm i -D html-webpack-plugin
```

在`webpack.config.js`中加入插件：

```js
const path = require("path");
// 引入模块
const HtmlWebpackPlugin =require("html-webpack-plugin");
const dist = path.join(__dirname, '..', 'dist');

module.exports = {
    entry: {
        main: './src/home.js',
    },
    output: {
        path: dist,
        filename: '[name].[chunkHash:8].js'
    },
    // 引入插件
    plugins: [
            new HtmlWebpackPlugin()
    ]
};
```

进行`npm run build`，在`dist`中生成了`index.html`，其中使用的js文件是打包的js文件。

### html模版

使用html模版则需要在项目目录中建立`public`文件夹，在其中建立`home.html`，如下：

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>自动生成模版</title>
</head>
<body>
<div id="root"></div>
</body>
</html>
```

修改`webpack.config.js`，为插件添加配置：

```js
const path = require("path");
const HtmlWebpackPlugin =require("html-webpack-plugin");
const dist = path.join(__dirname, '..', 'dist');

module.exports = {
    entry: {
        main: './src/home.js',
    },
    output: {
        path: dist,
        filename: '[name].[chunkHash:8].js'
    },
    plugins: [
            new HtmlWebpackPlugin({
                title: "模版测试",
                template: "public/home.html"
            })
    ]
};
```

可以使用模版语法来定制生成的html。

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title><%= htmlWebpackPlugin.options.title %></title>
</head>
<body>
<div id="root"></div>
</body>
</html>
```

再次打包则生成的html如下：

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>模版测试</title>
</head>
<body>
<div id="root"></div>
<script type="text/javascript" src="main.299861a9.js"></script></body>
</html>
```

### css

安装`style-loader`和`css-loader`。`style-loader`用于插入css到js里。`css-loader`用于处理css。

```
npm i -D style-loader css-loader
```

修改`webpack.config.js`，添加loader：

```js
const path = require("path");
const HtmlWebpackPlugin =require("html-webpack-plugin");
const dist = path.join(__dirname, '..', 'dist');

module.exports = {
    entry: {
        main: './src/home.js',
    },
    output: {
        path: dist,
        filename: '[name].[chunkHash:8].js'
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [ 'style-loader', 'css-loader' ]
            }
        ]
    },
    plugins: [
            new HtmlWebpackPlugin({
                title: "模版测试",
                template: "public/home.html"
            })
    ]
};
```

`loader`的执行顺序从后向前。

在js文件中引入css。

```js
import './main.css';
console.log("home");
```

此时进行打包，css会进入js并在执行的时候生效。

如果希望提取css到单独文件，则需要安装`mini-css-extract-plugin`插件。

```
npm i -D mini-css-extract-plugin
```

修改`webpack.config.js`，加入该插件的配置，修改位置已标出。

```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
// [1]
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

const dist = path.join(__dirname, '..', 'dist');

module.exports = {
    entry: {
        main: './src/home.js',
    },
    output: {
        path: dist,
        filename: '[name].[chunkHash:8].js'
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                // [2]
                use: [ MiniCssExtractPlugin.loader, 'css-loader' ]
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            title: "模版测试",
            template: "public/home.html"
        }),
        // [3]
        new MiniCssExtractPlugin({
            filename: '[name].css'
        })
    ]
};
```

进行打包时css文件会单独到处，并在html中引入。

### 构建导出目录结构

修改`webpack.config.js`，直接在导出部分添加路径即可。

```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

const dist = path.join(__dirname, '..', 'dist');

module.exports = {
    entry: {
        main: './src/home.js',
    },
    output: {
        path: dist,
        // [1]
        filename: 'js/[name].[chunkHash:8].js'
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [ MiniCssExtractPlugin.loader, 'css-loader' ]
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            title: "模版测试",
            template: "public/home.html"
        }),
        new MiniCssExtractPlugin({
            // [2]
            filename: 'css/[name].[chunkHash:8].css'
        })
    ]
};
```

### 配置开发服务器

安装插件`webpack-dev-server`。

```
npm i -D webpack-dev-server
```

修改`package.json`，添加`dev`命令。

```json
{
  "name": "test-react-2",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build": "webpack --mode production --config scripts/webpack.config.js",
    "dev": "webpack-dev-server --mode development --config scripts/webpack.config.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "css-loader": "^3.2.0",
    "html-webpack-plugin": "^3.2.0",
    "mini-css-extract-plugin": "^0.8.0",
    "style-loader": "^1.0.0",
    "webpack": "^4.41.2",
    "webpack-cli": "^3.3.9"
  }
}
```

修改`webpack.config.js`，可以配置端口号等。

```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

const dist = path.join(__dirname, '..', 'dist');

module.exports = {
    entry: {
        main: './src/home.js',
    },
    output: {
        path: dist,
        filename: 'js/[name].[chunkHash:8].js'
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [ MiniCssExtractPlugin.loader, 'css-loader' ]
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            title: "模版测试",
            template: "public/home.html"
        }),
        new MiniCssExtractPlugin({
            filename: 'css/[name].[chunkHash:8].css'
        })
    ],
    // [1]
    devServer: {
        port: 3000,
        open: true
    }
};
```

### 配置css预处理器

安装`less`和`less-loader`。

```
npm i -D less less-loader
```

修改`webpack.config.js`，添加less文件规则。

```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

const dist = path.join(__dirname, '..', 'dist');

module.exports = {
    entry: {
        main: './src/home.js',
    },
    output: {
        path: dist,
        filename: 'js/[name].[chunkHash:8].js'
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [ MiniCssExtractPlugin.loader, 'css-loader' ]
            },
            // [1]
            {
                test: /\.less$/,
                use: [ MiniCssExtractPlugin.loader, 'css-loader', 'less-loader' ]
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            title: "模版测试",
            template: "public/home.html"
        }),
        new MiniCssExtractPlugin({
            filename: 'css/[name].[chunkHash:8].css'
        })
    ],
    devServer: {
        port: 3000,
        open: true
    }
};
```

添加一个less文件并引入。

```less
body {
  color: aliceblue;
}
```

```js
import './main.css'
// [1]
import './test.less'
console.log("home");
```

