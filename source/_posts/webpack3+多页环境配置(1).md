---
title: webpack3+多页环境配置(1)
date: 2018-04-27 13:03:53
categories: Javascript
tags: webpack
---

# webpack3+多页环境配置(1)

#### 什么是webpack?
***

在我看来就是一个前端开发工具，至于为什么要使用？一个字懒！越懒越方便，越懒越简单。至于为什么要用3+不用最新的webpack4，只是目前公司环境下都是3+。
下面就简单的引用下官网介绍：

> 本质上，webpack 是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。

直接从官网抠图看更直接:
![webpack1_1](/images/webpack1_1.png)

#### 为什么要搭建多页
***

webpack说实话用在spa上是比较多的，特别是现在vue、react都是依然webpack。但是有时候写个静态页面，写个官网，写几个不复杂的页面，往往不需要使用框架，特别是官网，还需要seo优化。然后webpack用习惯了，难免想使用less、sass、压缩、base64等等。
总而言之，为了懒，还是配个环境一劳永逸-. -

#### 安装webpack
***

新建一个文件夹，命名为demo。打开命令行工具，敲入:
```
npm init
```
中间一路回车就行，目的就是为了生成`package.json`文件，后期内容都是可以编辑的。
然后局部安装webpack，全局安装可能会对别的项目造成影响，当你同时开发多个项目的时候。
这里指定安装3.8.1的版本，只是为了和github上demo中的webpack同步。
```
npm install --save-dev webpack@3.8.1
```

#### 创建基本项目结构
***

在demo文件夹下建立src文件夹、config文件夹和static文件夹；建立.gitignore和.babelrc文件。

* src文件夹：用来存放项目代码
* config文件夹：用来存放webpack配置代码
* .gitignore文件：git忽略文件配置
* .babelrcg文件：babel的配置文件，在这里主要配置使用es6

比写代码还累，基本目录结构好了，接下来就是安装各种依赖了~~



