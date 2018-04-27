---
title: webpack3+多页环境配置(3)
date: 2018-04-27 17:13:38
categories: Javascript
tags: webpack
---

# webpack3+多页环境配置(3)

#### 配置webpack.config.prod
***

`webpack.config.prod.js`是生产配置文件。在`config`文件夹下新建`webpack.config.prod.js`文件。
`webpack.config.prod.js`: 
```
const webpackBase = require('./webpack.config.base.js');
const config = require('./config.js');
const webpack = require('webpack');
const webpackMerge = require('webpack-merge');
const webpackProd = {
  output: {
    filename: 'js/[name].[chunkhash:8].bundle.js', // 生产环境用chunkhash
  },
  plugins: [
    new webpack.DefinePlugin({ // 指定为生产环境，进而让一些library可以做一些优化
      'process.env.NODE_ENV': JSON.stringify('production')
    }),
    new webpack.HashedModuleIdsPlugin(), // 生产环境用于标识模块id
  ]
};
module.exports = webpackMerge(webpackBase, webpackProd);
```
输入`npm run build`可以看见在demo下生成了dist文件夹，证明打包成功了，可以直接打开index.html查看。

#### 配置多页面的项目结构
***

首先在`src`下创建`html`文件夹用来存放页面；这里注意，我们以官网首页为例，我们打开官网后直接定位找到的应该是index.html，如果我们把index.html和其他页面放在`html`一起，就不太好，所以把index.html单独作为入口提到外面。
我们先在`html`下创建`page1.html`和`page2.html`;并分别在`js`下创建对应的js入口文件。结构如下：

```
├─index.html # 主页面入口
    ├─html # 各个子页面
    │  ├─page1.html # page1页面
    |  ├─page2.html # page2页面
    │  └─....
    ├─js # js文件
    |  ├─index.js # index js
    |  ├─page1.js # page1 js
    |  ├─page2.js # page2 js
```

#### 配置多页面的入口
***

修改`webpack.config.base.js`配置，把page打包到html文件夹中:

```
const page1Cfg = {
  filename: 'html/page1.html',
  template: path.join(config.HTML_PATH, './page1.html'),
  chunks: ['page1']
};
Entries.page1 = './src/js/page1.js';
HTMLPlugins.push(new HTMLWebpackPlugin(page1Cfg));

const page2Cfg = {
  filename: 'html/page2.html',
  template: path.join(config.HTML_PATH, './page2.html'),
  chunks: ['page2']
};
Entries.page2 = './src/js/page2.js';
HTMLPlugins.push(new HTMLWebpackPlugin(page2Cfg));
```

执行`npm run dev/build`可以看到基本的多页已经配置完成了。

#### 配置多页面的入口之我很懒
***

多页多页就是很多页面，每个页面配置一遍，麻烦死了。直接搞个函数自动读取好了。要自动读取的话要引入node的`fs`来读取文件：

```
const fs = require('fs');
```

然后封装一个自动获取html文件方法：

```
const getFileNameList = (path) => {
  let fileList = [];
  let dirList = fs.readdirSync(path);
  dirList.forEach(item => {
    if (item.indexOf('html') > -1) {
      fileList.push(item.split('.')[0]);
    }
  });
  return fileList;
};
let htmlDirs = getFileNameList(config.HTML_PATH);
```

接下来类似对`index.html`的处理，对`htmlDirs`遍历，用`html-webpack-plugin`生成对应的html：

```
htmlDirs.forEach((page) => {
  let htmlConfig = {
    filename: `html/${page}.html`,
    template: path.join(config.HTML_PATH, `./${page}.html`), // 模板文件
    chunks: [page]
  };
  Entries[page] = `./src/js/${page}.js`;

  const htmlPlugin = new HTMLWebpackPlugin(htmlConfig);
  HTMLPlugins.push(htmlPlugin);
});
```

再次执行dev/build可以看到都成功了。至此多页就算基本配置完成了，接下来就是各种优化咯。
这里贴上此次base代码：

```
const webpack = require('webpack'); 
const path = require('path');
const config = require('./config.js');
const fs = require('fs');
const HTMLWebpackPlugin = require("html-webpack-plugin");
const getFileNameList = (path) => {
  let fileList = [];
  let dirList = fs.readdirSync(path);
  dirList.forEach(item => {
    if (item.indexOf('html') > -1) {
      fileList.push(item.split('.')[0]);
    }
  });
  return fileList;
};
let htmlDirs = getFileNameList(config.HTML_PATH);

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
htmlDirs.forEach((page) => {
  let htmlConfig = {
    filename: `html/${page}.html`,
    template: path.join(config.HTML_PATH, `./${page}.html`), // 模板文件
    chunks: [page]
  };
  Entries[page] = `./src/js/${page}.js`;

  const htmlPlugin = new HTMLWebpackPlugin(htmlConfig);
  HTMLPlugins.push(htmlPlugin);
});
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