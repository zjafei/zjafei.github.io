---
title: vuepie----基于apicloud的vue混和app开发脚手架
date: 2018-08-19 20:37:27
tags: ['vue','apicloud','hybrid app','tools','cli']
category: 'coding'
---

![Vuepie logo](https://raw.githubusercontent.com/zjafei/vuepie/master/vuepieLogo.fw.png)
## 为什么做这个派？

> apicloud是很好的混合app开发框架，原生api支持丰富。但是apicloud只是完成了混合开发的业务，对于具体页面没有实现与nodejs体系的融合，导致了开发人员疲于奔命的开发业务组件。所以如何可以方便的使用npm大量的免费组件，大大降低了业务开发的成本是个问题。`vuepie`就是为解决这个问题而产生的。使用`vuepie`你可以想做常规的vue页面那样去开发apicloud混和app。就是原有apicloud的都在，npm的还可以用。 

<!--more-->

## 安装

### 下载vuepie
```bash
git clone git@github.com:zjafei/vuepie.git
```

### 安装依赖包
```bash
npm i
```

### 启动开发环境
```bash
npm start
```

### 编译产出
```bash
npm run pro
```

### 真机调试和发布
> 同步 `dist` 目录下的所有文件即可。

## 脚手架目录结构

```
/root
 ├─ /dist              // 产出目录
 ├─ /src
 │   ├─ /assets        // 公共静态资源
 │   │   ├─ /css       // 样式文件
 │   │   └─ /images    // 图片文件
 │   ├─ /components    // vue 组件
 │   ├─ /utils         // 工具方法
 │   ├─ /service
 │   │   ├─ index.js   // api 请求
 │   │   └─ mock.js    // 数据 mock
 │   └─ /views         // 工具方法
 │       └─ /namespace // 页面
 ├─ .babelrc           // 代码风格配置
 ├─ .editorconfig      // 代码风格配置
 ├─ .eslintrc          // eslint 配置
 ├─ .gitignore         // git 忽略配置
 ├─ .postcssrc.js      // 样式后处理器配置
 ├─ config.xml         // apicloud app 配置
 ├─ package.json       // 项目配置
 ├─ README.md          // 项目说明
 └─ webpack.config.js  // 产出配置
```
## namespace页面目录结构和说明
```
/namespace          // 命名空间
 ├─ app.html       // 页面模板（一般不用编辑）
 ├─ app.js         // 入口 js（一般不用编辑）
 └─ app.vue        // 页面业务
```
> 一个views目录下namespace目录为命名空间目录，命名空间目录必须有app.html，app.js和app.vue文件，目录会在产出后生成对应namespace.html，namespace.js和namespace.css。例如我有一个如下目录结构的：

```
/src
 └─ /views
     └─ /index
         ├─ app.html
         ├─ app.js
         ├─ app.vue
         └─ /aboutUs
             ├─ app.html
             ├─ app.js
             └─ app.vue
```
> 那么执行编译产出后views目录下的文件出结果如下：

```
/dist
 ├─ /index
 │   ├─ aboutUs.css
 │   ├─ aboutUs.js
 │   └─ aboutUs.html
 ├─ index.css
 ├─ index.js
 └─ index.html
```
## 结语

> 欢迎小伙伴们使用。有需求建议和问题尽管提交issue，我尽力修正。当然直接pull request那是更好。