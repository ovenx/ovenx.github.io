---
title: 搭建 Webpack4 和 Vue2 项目
date: 2020-02-11 16:54:00
categories:
- 程序世界
tags:
- webpack
- vue
---
本文的环境：

* vue 2.6.11
* webpack 4.41.5
* babel 7.8.4
* node 12.13.1

### 安装 webpack
```bash
npm webpack webpack-cli -g
```

### 初始化项目
1. 创建目录，初始化
```
mkdir webpack4-vue2
cd webpack4-vu2
npm init 
```
2. 安装 webpack-dev-server
```bash
npm install webpack webpack-dev-server --save-dev
```
1. 新建 webpack.config.js
```bash
var path = require('path');
var webpack = require('webpack');
module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, './dist'),
    publicPath: '/dist/',
    filename: 'build.js'
  },
  devServer: {
    historyApiFallback: true,
    overlay: true
  },
  module: {
    rules: []
  }
}
```

### 配置 babel
babel 的目的是为了将 ES6 转为 ES5 语法，以便能够运行在当前和旧版本的浏览器或其他环境中。[参考文档](https://www.babeljs.cn/docs/)

1. 首先安装 babel，不同的环境可以 [参考文档](https://www.babeljs.cn/setup#installation)
``` bash
npm install @babel/core @babel/preset-env --save-dev

```
2. 安装 transform-runtime 插件 [参考文档](https://www.babeljs.cn/docs/babel-plugin-transform-runtime)
```bash
npm install @babel/plugin-transform-runtime --save-dev
npm install @babel/runtime --save 
```

3. 项目根目录新建 .babelrc 文件
```bash
{
  "plugins": [
    "@babel/plugin-transform-runtime"
  ],
  "presets": [
    "@babel/preset-env"
  ]
}
```

到这里 babel 基本配置完成，那么在 webpack 中如何使用 babel 呢，我们需要用到 babel-loader，[参考文档](https://www.webpackjs.com/loaders/babel-loader/)

1. 安装 babel-loader
```bash
npm install babel-loader --save-dev
```

2. webpack.config.js 添加一个 loader
```bash
{
    test: /\.js$/,
    loader: 'babel-loader',
    exclude: /node_modules/  
}
```

### 配置 vue 

1. 安装vue 及相关组件
```
npm install vue --save
npm install vue-loader vue-template-compiler --save-dev
npm install node-sass css-loader vue-style-loader sass-loader --save-dev // 解析样式
npm install file-loader --save-dev // 解析静态资源

```
2. 配置 webpack.config.js，添加相关 loader
```bash
{
  test: /\.css$/,
  use: [
    'vue-style-loader',
    'css-loader'
  ],
},
{
  test: /\.scss$/,
  use: [
    'vue-style-loader',
    'css-loader',
    'sass-loader'
  ],
},
{
  test: /\.sass$/,
  use: [
    'vue-style-loader',
    'css-loader',
    'sass-loader?indentedSyntax'
  ],
},
{
  test: /\.(png|jpg|gif|svg)$/,
  loader: 'file-loader',
  options: {
    name: '[name].[ext]?[hash]',
    esModule: false,
    publicPath: '../dist/images/',
    outputPath: 'images/'
  }
},
{
  test: /\.(html|htm)$/,
  use: [
    {
    loader: 'html-withimg-loader',
    }
  ]
},
{
  test: /\.vue$/,
  loader: 'vue-loader',
  options: {
    loaders: {
    'scss': [
      'vue-style-loader',
      'css-loader',
      'sass-loader'
    ],
    'sass': [
      'vue-style-loader',
      'css-loader',
      'sass-loader?indentedSyntax'
    ]
  }  
}
```

3. 配置 webpack.config.js 添加 vue，vue-loader 的支持
```bash
const VueLoaderPlugin = require('vue-loader/lib/plugin')
plugins: [
  new VueLoaderPlugin()
],
resolve: {
  alias: {
    'vue$': 'vue/dist/vue.esm.js'
  }
}
```
4. 新建文件 

src/main.js
```bash
import Vue from 'vue';
import App from './App.vue';

Vue.config.productionTip = false

new Vue({
  el: '#app',
  template: '<App/>',
  components: { App }
})
```

src/App.vue
```bash
<template>
  <div id="app">
    <h1>{{ msg }}</h1>
    <input type="text" v-model="msg" />
    <img src="./images/logo.png" />
  </div>
</template>

<script>
export default {
  name: "app",
  data() {
    return {
      msg: "default text data"
    };
  },
  created() {
    this.getData();
  },
  methods: {
    async getData() {
      const promiseData = function() {
        return new Promise((resolve, reject) => {
          resolve("changed text data");
        });
      };
      this.msg = await promiseData();
    }
  }
};
</script>

<style lang="scss">
#app {
  img {
    margin-top: 20px;
    width: 200px;
  }
}
</style>
```

src/index.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>webpback4-vue2</title>
</head>

<body>
  <div id="app"></div>
  <script src="/dist/build.js"></script>
</body>

</html>
```

### 运行

1. package.json 中添加 script
```bash
"scripts": {
  "dev": "webpack-dev-server --open --hot",
  "build": "webpack --progress --hide-modules"
},
```

2. 运行
```bash
npm run dev
```

3. 打包
```
npm run build
```

### 结束语
以上就是一个简单的webpack4 + vue2 demo项目搭建的全过程。本文仅作参与，如果错误或疏漏，欢迎大家批评指正。源码链接 [https://github.com/ovenx/webpack4-vue2](https://github.com/ovenx/webpack4-vue2)