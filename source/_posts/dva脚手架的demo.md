---
title: dva脚手架的demo
date: 2017-12-26 20:52:27
tags: ['reduxe','redux-saga','effect','dispatch','reducer']
category: 'coding'
---

# dva-demo
![D.VA](/0/D.Va.jpg)
## 为什么使用dva
*  reducer, saga, action 都是分离的，编辑成本高，需要在 reducer, saga, action 之间来回切换
* 不便于组织业务模型 (或者叫 domain model)，需要复制很多文件。
* dva是 framework，很明确地告诉你每个部件应该怎么写。
<!--more-->

```javascript
app.model({
  namespace: 'demo',
  state: {
  },
  effects: {
  },
  reducers: {
  },
});
```

介绍 model 的 key 与原来的 redux-saga 的关系 ：
* namespace - 对应 rootReducer 时的 key 值
* state - 对应 reducer 的 initialState
* effects - 对应 saga
* reducers 

## 实战项目
### 文件目录结构
```bash
/root
  ├─ dist/              // 产出目录
  ├─ mock/              // mock 数据
  ├─ public/            // 模板和静态资源
  ├─ src
  │   ├─ assets/        // 静态资源
  │   ├─ components/    // 组件
  │   ├─ models/        // 业务模型
  │   ├─ routes/        // 页面组件
  │   ├─ services/      // 服务
  │   ├─ utils/         // 工具方法
  │   ├─ index.css      // 样式
  │   ├─ index.ejs      // spa 模板
  │   ├─ index.js       // spa 入口
  │   └─ router.js      // 路由文件
  ├─ .editorconfig      // 代码风格配置
  ├─ .eslintrc          // eslint 配置
  ├─ .gitignore         
  ├─ .roadhogrc         // roadhog 配置
  ├─ .roadhogrc.mock.js // roadhog 数据mock
  └─ package.json       // 项目配置
```
### 什么是 roadhog
![roadhog](/0/roadhog.png)

基于webpack的cli工具，分别用于本地调试和构建，并且提供了特别易用的 mock 功能。

### 代码实战
1. 安装dva，antd 和 babel-plugin-import

```bash
npm install dva-cli -g
```
```bash
dva new dva-quickstart
```
```bash
cd dva-quickstart
npm start    
```

几秒钟后，你会看到以下输出：

```bash
Compiled successfully!

The app is running at:

  http://localhost:8000/

Note that the development build is not optimized.
To create a production build, use npm run build.
```

在浏览器里打开 http://localhost:8000 ，你会看到 dva 的欢迎界面。

通过 npm 安装 antd 和 babel-plugin-import 。babel-plugin-import 是用来按需加载 antd 的脚本和样式的。

```bash
npm install antd babel-plugin-import --save
```

编辑 .roadhogrc，使 babel-plugin-import 插件生效。

```diff
  "extraBabelPlugins": [
-    "transform-runtime"
+    "transform-runtime",
+    ["import", { "libraryName": "antd", "libraryDirectory": "es", "style": "css" }]
  ],
```

2. 创建page组件
/root/src/routes/demo/index.js 的代码如下

```JSX
import React from 'react';
import { connect } from 'dva';

class Demo extends React.Component {
    static ROUTE = '/';
    
    render() {
        return (
            <div>hello</div>
        );
    }
}

export const DemoClass = Demo;

export default connect()(Demo);         
```   

/root/src/router.js 的代码如下

```JSX
import React from 'react';
import { Router, Route } from 'dva/router';
import Demo, { DemoClass } from './routes/Demo';

function RouterConfig({ history }) {
  return (
    <Router history={history}>
      <Route path={DemoClass.ROUTE} component={Demo} />
    </Router>
  );
}

export default RouterConfig;       
``` 

会看到 hello

2. 创建业务模型
/root/src/models/demo.js 的代码如下

```javascript
const namespace = 'demo';

export default {
    namespace,
    state: {
        showText:'hello demo',
    },
    reducers: {
        overrideStateProps(state, { payload }) {
            return {
                ...state,
                ...payload,
            };
        },
        updateStateProps(state, { payload }) {
            const { name, value } = payload;
            return {
                ...state,
                ...{ [name]: { ...state[name], ...value } },
            };
        },

    },
    effects: {
    },
}
```

/root/src/index.js 的代码如下

```javascript
import dva from 'dva';
import './index.css';

const app = dva();

app.model(require('./models/demo'));

app.router(require('./router'));

app.start('#root');
```

3. 绑定state 

/root/src/routes/demo/index.js 的代码如下

```javascript
import React from 'react';
import { connect } from 'dva';

class Demo extends React.Component {
    static ROUTE = '/';

    render() {
        const { showText, dispatch } = this.props;
        return (
            <div>{showText}</div>
        );
    }
}

export const DemoClass = Demo;

export default connect(({ demo }) => ({
    showText: demo.showText,
}))(Demo);
```

4. 添加reduce 

/root/src/routes/demo/index.js 的代码如下

```javascript
import React from 'react';
import { connect } from 'dva';
import { Button } from 'antd';

class Demo extends React.Component {
    static ROUTE = '/';

    render() {
        const { showText, dispatch } = this.props;
        return (
            <div>
                <h1>{showText}</h1>
                <Button
                    onClick={() => {
                        dispatch({
                            type: 'demo/overrideStateProps',
                            payload: {
                                showText: showText + 1,
                            },
                        });
                    }}
                >
                    点击
                </Button>
            </div>
        );
    }
}

export const DemoClass = Demo;

export default connect(({ demo }) => ({
    showText: demo.showText,
}))(Demo);
```

/root/src/models/demo.js 的代码如下

```javascript
const namespace = 'demo';

export default {
    namespace,
    state: {
        showText: 0,
    },
    reducers: {
        overrideStateProps(state, { payload }) {
            return {
                ...state,
                ...payload,
            };
        },
        updateStateProps(state, { payload }) {
            const { name, value } = payload;
            return {
                ...state,
                ...{ [name]: { ...state[name], ...value } },
            };
        },

    },
    effects: {
    },
};
```

5. 添加 effects

/root/src/utils/request.js 的代码如下
```javascript
import fetch from 'dva/fetch';

function parseJSON(response) {
  return response.json();
}

function checkStatus(response) {
  if (response.status >= 200 && response.status < 300) {
    return response;
  }

  const error = new Error(response.statusText);
  error.response = response;
  throw error;
}

export default function request(url, options) {
  return fetch(url, options)
    .then(checkStatus)
    .then(parseJSON)
    .then(data)
    .catch(err => ({ err }));
}
```

/root/src/services/Demo.js 的代码如下
```javascript
import request from '../utils/request';

export default class Demo {
    static get() {
        return request('/api/users', { credentials: 'include' });
    }
}
```

/root/src/models/demo.js 的代码如下
```javascript
import Demo from '../services/Demo'

const namespace = 'demo';

export default {
    namespace,
    state: {
        showText: 0,
    },
    reducers: {
        overrideStateProps(state, { payload }) {
            return {
                ...state,
                ...payload,
            };
        },
        updateStateProps(state, { payload }) {
            const { name, value } = payload;
            return {
                ...state,
                ...{ [name]: { ...state[name], ...value } },
            };
        },

    },
    effects: {
        *get(action, { call }) {
            const req= yield call(Demo.get);
            console.log(req);
        },
    },
};
```