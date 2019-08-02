---
title: taro with DVA ----tarojs 配合 dva.js的实践
date: 2019-08-02 11:22:32
tags: ['taro.js','js','javascript','react','小程序','dvajs','最佳实践']
category: 'coding'
---

![taro with DVA ](/0/taro_dva.jpg)

## 为什么使用 tarojs
Taro 是一套遵循 React 语法规范的 多端开发 解决方案。现如今市面上端的形态多种多样，Web、React-Native、微信小程序等各种端大行其道，当业务要求同时在不同的端都要求有所表现的时候，针对不同的端去编写多套代码的成本显然非常高，这时候只编写一套代码就能够适配到多端的能力就显得极为需要。
使用 Taro，我们可以只书写一套代码，再通过 Taro 的编译工具，将源代码分别编译出可以在不同端（微信/百度/支付宝/字节跳动/QQ小程序、快应用、H5、React-Native 等）运行的代码。

## 为什么使用 davjs
因为 Taro 是类 React 框架，所以　React　有的问题　Taro　也同样存在。所以 davjs 的介入就变的十分重要。（具体dvajs做了什么可以看文章 [dva脚手架的demo](https://zjafei.coding.me/2017/12/26/dva%E8%84%9A%E6%89%8B%E6%9E%B6%E7%9A%84demo/)）

## 实践demo的目的

* 规范项目目录结构
* 总结一些常用的工具和服务
* 提供常规业务的最佳实践

<!--more-->

## 相关框架内容请去官网

* [taro](https://github.com/NervJS/taro)
* [dvajs](https://github.com/dvajs/dva)

## 使用 npm 或者 yarn 全局安装

```
npm install -g @tarojs/cli
// 或
yarn global add @tarojs/cli
```

### clone 项目
```bash
git clone git@github.com:zjafei/taro-dva-demo.git
```

## 然后使用npm 或者yarn 安装依赖

```
npm install
// 或
yarn
```

## 启动小程序开发环境

```
npm run dev:weapp
// 或
yarn run dev:weapp
```
*微信开发者工具导入项目根目录进行预览*

## 编译产出小程序

```
npm run build:weapp
// 或
yarn run build:weapp
```

## 创建页面

```
npm run page 'pageDemo'
// 或
yarn run page 'pageDemo'
```

### assets目录下的静态资源的上传
```
./deploy scp 
```

## 项目结构
```bash
/
├── config
│   ├── dev.js                      // 开发环境的配置
│   ├── index.js                    // 配置
│   └── pro.js                      // 生产环境的配置             
├── src
│   ├── assets                      // 静态资源
│   ├── components                  // 公用组件
│   ├── models                      // 模型
│   │   ├── globalModel.js          // 全局模型
│   │   └── index.js                // 所有模型
│   ├── pages
│   │   └── PageDemo
│   │       ├── index.jsx           // 业务页面
│   │       ├── index.scss          // 页面样式
│   │       └── model.js            // 页面模型
│   ├── server
│   │   ├── api.js                  // api
│   │   ├── hostMap.js              // baseUrl的配置
│   │   └── mock.js                 // mock 数据
│   ├── utils                       // 工具方法
│   ├── app.global.scss             // 全局公共样式 样式名称不会被 CSS Modules
│   ├── app.js                      // 程序入口
│   ├── base.scss                   // 样式变量
│   └── index.html                  // spa页面的模板
├── .editorconfig                   
├── .eslintignore                   
├── .eslintrc                       
├── .gitignore                      
├── .prettierignore                 
├── .prettierrc                     
├── .stylelintrc.json               
├── componentsTemplate.js           
├── deploy                          // 脚本文件
├── package.json                    
├── pageTemplate.js                 // 页面模板脚本
├── project.config.json             
├── README.md                       
├── tsconfig.json                   
└── tslint.json                     
```

## 结语

> 欢迎小伙伴们使用。有需求建议和问题尽管提交issue，我尽力修正。当然直接pull request那是更好。