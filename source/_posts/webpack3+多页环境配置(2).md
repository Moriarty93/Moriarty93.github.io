---
title: webpack3+多页环境配置(2)
date: 2018-04-27 15:07:06
categories: Javascript
tags: webpack
---

# webpack3+多页环境配置(2)

#### 区分开发和生产环境
***

开发和生产环境的某些配置是不一样了，这时候最好是分为2个配置文件，当开发时启动开发配置`webpack.config.dev.js`，打包时启动生产配置`webpack.config.pro.js`。这样区分不仅在后期更好维护，在开发时候也更加快速，不用进行无用的修改。
具体修改如下：
```
"scripts": {
    "build": "webpack  --progress --color --config ./config/webpack.config.prod.js",
    "dev": "webpack-dev-server --progress --color --open --config ./config/webpack.config.dev.js"
  }
```

#### 配置入口文件
***

在`src`下新建`index.html`文件作为项目入口文件，同时在`src`下建立`js`文件夹，然后创建`index.js`文件作为识别。

index.html:
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
</head>
<body>
  <div class="wrap">
    Moriarty
  </div>
</body>
</html>
```
index.js:
```
console.log('Moriarty');
```

#### 配置config.js
***

`config.js`是对全局路径的配置。在`config`文件夹下创建`config.js`文件：
```
const path = require('path');
const PROJECT_PATH = process.cwd();
const config = {
  PROJECT_PATH,  // 项目目录
  CONFIG_PATH: path.join(__dirname),  // 配置文件目录
  BUILD_PATH: path.join(PROJECT_PATH, './dist/'), // 打包目录
  SRC_PATH: path.join(PROJECT_PATH, './src/'),  // 源文件目录
  PUBLIC_PATH: path.join(PROJECT_PATH, './dist/static/'), // 静态文件存放目录
  HTML_PATH: path.join(PROJECT_PATH, './src/html/'),
  NODE_MODULES_PATH: path.join(PROJECT_PATH, './node_modules/'), // node_modules目录
};
module.exports = config;
```

#### 配置webpack.config.base
***

`webpack.config.base.js`是开发和生产共有的基本配置文件。在`config`文件夹下新建`webpack.config.base.js`文件。同时安装`html-webpack-plugin`用于生成html。
```
npm install --save-dev html-webpack-plugin
```

webpack.config.base.js文件内容如下，其中对js的处理只包括src下的文件。
```
const webpack = require('webpack'); 
const path = require('path');
const HTMLWebpackPlugin = require("html-webpack-plugin");
let HTMLPlugins = []; // 保存HTMLWebpackPlugin实例
let Entries = {}; // 保存入口列表
// 单独处理项目首页 ---结构不同
const indexCfg = {
  filename: 'index.html',
  template: path.join(config.SRC_PATH, 'index.html'),
  chunks: ['index']
};
Entries.index = './src/js/index.js';
HTMLPlugins.push(new HTMLWebpackPlugin(indexCfg));
module.exports = {
  context: config.PROJECT_PATH,
  entry: Entries,
  output: {
    path: config.BUILD_PATH, 
  },
  plugins: [
    ...HTMLPlugins
  ]
};
```

#### 配置webpack.config.dev
***

`webpack.config.dev.js`是开发配置文件。在`config`文件夹下新建`webpack.config.dev.js`文件。同时安装`webpack-dev-server`、`webpack-merge`，分别用于本地开发和合并webpack设置。这里注意`webpack-dev-server`较高的版本会报错！
```
npm install --save-dev webpack-dev-server@2.9.4 webpack-merge
```
`webpack.config.dev.js`: 
```
const webpack = require('webpack');
const webpackBase = require('./webpack.config.base.js');
const config = require('./config.js');
const webpackMerge = require('webpack-merge');
const webpackDev = {
  output: {
    filename: 'js/[name].[hash:8].bundle.js', // 开发环境用hash
  },
  devtool: 'cheap-module-eval-source-map', // 开发环境设置sourceMap，生产环境不使用
  devServer: { // 启动devServer，不会在本地生成文件，所有文件会编译在内存中(读取速度快)
    contentBase: './dist/', // 这个目录下的内容可被访问
    overlay: true, // 错误信息直接显示在浏览器窗口中
    inline: true, // 实时重载的脚本被插入到你的包(bundle)中，并且构建消息将会出现在浏览器控制台
    hot: true, // 配合webpack.NamedModulesPlugin、webpack.HotModuleReplacementPlugin完成MHR
    host: "0.0.0.0", // 设置为0.0.0.0并配合useLocalIp可以局域网访问
    useLocalIp: true, // 使用本机IP打开devServer，而不是localhost
  },
  plugins: [
    new webpack.NamedModulesPlugin(), // 开发环境用于标识模块id
    new webpack.HotModuleReplacementPlugin() // 热替换插件
  ]
};
module.exports = webpackMerge(webpackBase, webpackDev);
```

到此输入`npm run dev`就可以看到浏览器自动打开并且页面显示Moriarty，打开控制台也看到输出Moriarty。开发的基本环境就算完成咯。