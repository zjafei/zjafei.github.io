---
title: WDS错误提示代码规范HMR的配置
date: 2017-12-30 10:28:54
tags: ['WDS','eslint','HMR','webpack-dev-server']
category: 'coding'
---
## 项目结构
```bash
/
├─ app
│   ├─ index.js
│   └─ print.js
├─ build
├─ package.json
└─ webpack.config.js
```
<!--more-->
/app/index.js

```javascript
import print from './print';

function component(text = 'hello world!') {
    const element = document.createElement('div');
    element.innerHTML = text;

    const p = document.createElement('input');
    p.type = 'checkbox';
    element.appendChild(p);

    const btn = document.createElement('button');
    btn.id = 'btn';
    btn.innerHTML = '点击';
    btn.onclick = print.log;
    element.appendChild(btn);

    return element;
}

document.body.appendChild(component());
```

/app/print.js

```javascript
export default {
    log() {
        console.log('5678');
    },
};
```

/package.json

```json
{
    "name": "webpack-dev-server-demo",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
    },
    "keywords": [],
    "author": "",
    "license": "ISC",
    "devDependencies": {
        "html-webpack-plugin": "^2.28.0",
        "webpack": "^2.3.2"
    }
}
```

/webpack.config.js

```javascript
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const PATHS = {
  app: path.join(__dirname, 'app'),
  build: path.join(__dirname, 'build'),
};

module.exports = {
  entry: {
    app: PATHS.app,
  },
  output: {
    path: PATHS.build,
    filename: '[name].js',
  },
  plugins: [
    new HtmlWebpackPlugin({
      title: 'Webpack demo',
    }),
  ],
};
```
## 初始化项目

```bash
npm install
```

## 安装 WDS----webpack-dev-server

```bash
npm install webpack-dev-server --save-dev
```

## 修改 package.json， 添加 script:

```diff
{
    "name": "webpack-dev-server-demo",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
-       "test": "echo \"Error: no test specified\" && exit 1"
+       "test": "echo \"Error: no test specified\" && exit 1",
+       "start": "webpack-dev-server --env development",
+       "build": "webpack --env production"
    },
    "keywords": [],
    "author": "",
    "license": "ISC",
    "devDependencies": {
        "html-webpack-plugin": "^2.28.0",
        "webpack": "^2.3.2",
        "webpack-dev-server": "^2.9.7"
    }
}
```
## 启动服务

```bash
npm start
```

浏览器中打开  [http://localhost:8080/](http://localhost:8080/)，修改 /app/print.js 代码，页面会整体刷新。

## WDS 端口号等配置相关

修改 /webpack.config.js

```diff
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

const PATHS = {
  app: path.join(__dirname, 'app'),
  build: path.join(__dirname, 'build'),
};

module.exports = {
+ devServer: {
+   host: process.env.HOST,
+   port: 8090,
+ },
  entry: {
    app: PATHS.app,
  },
  output: {
    path: PATHS.build,
    filename: '[name].js',
  },
  plugins: [
    new HtmlWebpackPlugin({
      title: 'Webpack demo',
    }),
  ],
};
```

```bash
npm start
```

浏览器中打开  [http://localhost:8090/](http://localhost:8090/)

## 配置 [ESLint](http://eslint.cn/) 实现代码规范自动测试
### 安装`ESLint`

```bash
npm install eslint --save-dev
```
### 修改 /package.json

```diff
{
  "name": "webpack-dev-server-demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "webpack-dev-server --env development",
-   "build": "webpack --env production"
+   "build": "webpack --env production",
+   "lintjs": "eslint app/ webpack.*.js --cache"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "eslint": "^4.14.0",
    "html-webpack-plugin": "^2.28.0",
    "webpack": "^2.3.2",
    "webpack-dev-server": "^2.9.7"
  }
}
```

### 添加ESLint配置文件 /.eslintrc.js

```javascript
module.exports = {
    env: {
        browser: true,
        commonjs: true,
        es6: true,
        node: true,
    },
    extends: 'eslint:recommended',
    parserOptions: {
        sourceType: 'module',
    },
    rules: {
        'comma-dangle': ['error', 'always-multiline'],
        indent: ['error', 4],
        'linebreak-style': ['error', 'unix'],
        quotes: ['error', 'single'],
        semi: ['error', 'always'],
        'no-unused-vars': ['warn'],
        'no-console': 0,
    },
};
```

### 执行ESLint

```bash
npm run lintjs
```

### 自动解决部分错误

```bash
npm run lintjs -- --fix
```

### 在webpage中自动进行eslint

安装 eslint-loader 

```bash
npm install eslint-loader --save-dev
```

在webpack.config.js添加相关配置

```diff
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

const PATHS = {
  app: path.join(__dirname, 'app'),
  build: path.join(__dirname, 'build'),
};

module.exports = {
  devServer: {
    host: process.env.HOST,
    port: 8090,
  },
  entry: {
    app: PATHS.app,
  },
  output: {
    path: PATHS.build,
    filename: '[name].js',
  },
+ module: {
+   rules: [
+     {
+       test: /\.js$/,
+       enforce: 'pre', // 前置loader
+       loader: 'eslint-loader',
+       options: {
+         emitWarning: true, // 是否发出警告
+       },
+     },
+   ],
+ },
  plugins: [
    new HtmlWebpackPlugin({
      title: 'Webpack demo',
    }),
  ],
};
```

修改一个js，运行

```bash
npm start
```

出现错误，运行

```bash
npm run lintjs -- --fix
```

接着运行

```bash
npm start
```

### 在浏览器中显示错误信息

在webpack.config.js添加相关配置

```diff
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

const PATHS = {
  app: path.join(__dirname, 'app'),
  build: path.join(__dirname, 'build'),
};

module.exports = {
  devServer: {
    host: process.env.HOST,
    port: 8090,
+   overlay: {
+     errors: true,
+     warnings: true,
+   },
  },
  entry: {
    app: PATHS.app,
  },
  output: {
    path: PATHS.build,
    filename: '[name].js',
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        enforce: 'pre',

        loader: 'eslint-loader',
        options: {
          emitWarning: true,
        },
      },
    ],
  },
  plugins: [
    new HtmlWebpackPlugin({
      title: 'Webpack demo',
    }),
  ],
};
```

### 添加 HMR(hot module replacement) 模块热更新

在webpack.config.js添加相关配置

```diff 
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
+ const webpack  = require('webpack');

const PATHS = {
  app: path.join(__dirname, 'app'),
  build: path.join(__dirname, 'build'),
};

module.exports = {
  devServer: {
    host: process.env.HOST,
    port: 8090,
    overlay: {
      errors: true,
      warnings: true,
    },
+   hotOnly: true,
  },
  entry: {
    app: PATHS.app,
  },
  output: {
    path: PATHS.build,
    filename: '[name].js',
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        enforce: 'pre',

        loader: 'eslint-loader',
        options: {
          emitWarning: true,
        },
      },
    ],
  },
  plugins: [
    new HtmlWebpackPlugin({
      title: 'Webpack demo',
    }),
+   new webpack.HotModuleReplacementPlugin(),
  ],
};
```

修改/app/index.js

```diff 
import print from './print';

function component(text = 'hello world!') {
    const element = document.createElement('div');
    element.innerHTML = text;

    const p = document.createElement('input');
    p.type = 'checkbox';
    element.appendChild(p);

    const btn = document.createElement('button');
    btn.id = 'btn';
    btn.innerHTML = '点击';
    btn.onclick = print.log;
    element.appendChild(btn);

    return element;
}

document.body.appendChild(component());

+if (module.hot) {
+    module.hot.accept('./print', () => {
+        console.log('print热更新了');
+        print.log();
+    });
+}
```

修改print.js console 有输出 但是点击按钮还是原来的输出

修改/app/index.js

```diff 
import print from './print';

function component(text = 'hello world!') {
    const element = document.createElement('div');
    element.innerHTML = text;

    const p = document.createElement('input');
    p.type = 'checkbox';
    element.appendChild(p);

    const btn = document.createElement('button');
    btn.id = 'btn';
    btn.innerHTML = '点击';
    btn.onclick = print.log;
    element.appendChild(btn);

    return element;
}

document.body.appendChild(component());

if (module.hot) {
    module.hot.accept('./print', () => {
        console.log('print热更新了');
+       window.document.getElementById('btn').onclick = print.log;
        print.log();
    });
}
```

修改print.js console 有输出 点击按钮是新的输出