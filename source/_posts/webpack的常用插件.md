---
title: webpack的常用插件
date: 2017-11-05 20:29:15
tags: ['webpack','前端','js','javascript']
type: 'coding'
---

__project__

``` diff
html-webpack-plugin
|- package.json
|- webpack.config.js
|- /src
  |- index.js
  |- style.css
|- /index.html
|- /node_modules
```

<!--more -->
__index.html__
``` html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>hello webpack</title>
    <meta name="description" content="">
    <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
<script src="./dist/index.js"></script>
</body>
</html>
```
__package.json__

``` javascript
{
  "name": "html-webpack-plugin",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build": "webpack --config webpack.config.js"
  },
  "repository": {},
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "css-loader": "^0.28.7",
    "style-loader": "^0.19.0",
    "webpack": "^3.8.1"
  }
}
```
__package.json__

``` javascript
const path = require('path');

module.exports = {
    entry: {
        index: './src/index.js',
    },
    output: {
        filename: '[name].js',
        path: path.resolve(__dirname, 'dist'),
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    { loader: 'style-loader' },
                    {
                        loader: 'css-loader',
                        options: {
                            modules: true
                        }
                    }
                ]
            }
        ]
    }
};
```


__src/index.js__

``` javascript
import style from './style.css';

const h1 = document.createElement('h1');
h1.className = style.danger;
h1.innerHTML = 'hello webpack';
document.body.appendChild(h1);
```

__src/style.css__

``` css
.danger{
    color: red
}
```

```bash
npm i
npm run build
```
__extract-text-webpack-plugin__

[extract-text-webpack-plugin](https://doc.webpack-china.org/plugins/extract-text-webpack-plugin/)：从捆绑包或捆绑包中提取文本到单独的文件中。

```bash
npm install --save-dev extract-text-webpack-plugin
```

``` diff 
const path = require('path');
+const ExtractTextWebpackPlugin = require('extract-text-webpack-plugin');

module.exports = {
    entry: {
        index: './src/index.js',
    },
    output: {
        filename: '[name].js',
        path: path.resolve(__dirname, 'dist'),
    },
    module: {
        rules: [
-           {
-               test: /\.css$/,
-               use: [
-                  { loader: 'style-loader' },
-                  {
-                       loader: 'css-loader',
-                       options: {
-                           modules: true
-                       }
-                   }
-               ]
-           }
+           {
+               test: /\.css$/,
+               use: ExtractTextWebpackPlugin.extract({
+                   fallback: "style-loader",
+                   use: [{
+                       loader: 'css-loader',
+                       options: {
+                           modules: true,
+                           localIdentName: '[path]_[name]_[local]_[hash:base64:5]'
+                       }
+                   }]
+               })
+           }
        ]
-   }
+   },
+plugins: [
+       new ExtractTextWebpackPlugin("style.css")
+   ]
};
```
__index.html__
``` diff
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>hello webpack</title>
    <meta name="description" content="">
    <meta name="viewport" content="width=device-width, initial-scale=1">
+   <link type="text/css" rel="stylesheet" href="./dist/style.css"/>
</head>
<body>
<script src="./dist/index.js"></script>
</body>
</html>
```
```bash
npm run build
```
```javascript
use: ExtractTextWebpackPlugin.extract(
    {
        fallback: "style-loader", // 编译后用什么loader来提取css文件
        use: "css-loader" // 指需要什么样的loader去编译文件,这里由于源文件是.css所以选择css-loader
    }
)

```
__clean-webpack-plugin__

[clean-webpack-plugin](https://www.npmjs.com/package/clean-webpack-plugin)：对目标文件的清理

```bash
npm install --save-dev clean-webpack-plugin
```

``` diff 
const path = require('path');
const ExtractTextWebpackPlugin = require('extract-text-webpack-plugin');
+const CleanWebpackPlugin = require('clean-webpack-plugin');

module.exports = {
    entry: {
        index: './src/index.js',
    },
    output: {
        filename: '[name].js',
        path: path.resolve(__dirname, 'dist'),
    },
    module: {
        rules: [
           {
               test: /\.css$/,
               use: ExtractTextWebpackPlugin.extract({
                   fallback: "style-loader",
                   use: [{
                       loader: 'css-loader',
                       options: {
                           modules: true,
                           localIdentName: '[path]_[name]_[local]_[hash:base64:5]'
                       }
                   }]
               })
           }
        ]
     },
plugins: [
+       new CleanWebpackPlugin(['dist']),
        new ExtractTextWebpackPlugin("style.css")
    ]
};
```
```bash
npm run build
```
```javascript
//参数说明
new CleanWebpackPlugin(
    ['dist/main.*.js','dist/manifest.*.js'],　  //匹配删除的文件
    {
        root: __dirname,       　　　　　　　　　　//根目录
        verbose:  true,        　　　　　　　　　　//开启在控制台输出信息
        dry:      false        　　　　　　　　　　//启用删除文件
    }
)
```

__html-webpack-plugin__
[html-webpack-plugin](https://doc.webpack-china.org/plugins/html-webpack-plugin)：简化了HTML文件的创建，以便为您的webpack包提供服务。 这对于在文件名中包含每次会随着变异会发生变化的哈希的webpack bundle尤其有用。

```bash
npm install --save-dev html-webpack-plugin
```
``` diff 
const path = require('path');
const ExtractTextWebpackPlugin = require('extract-text-webpack-plugin');
const CleanWebpackPlugin = require('clean-webpack-plugin');
+const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: {
        index: './src/index.js',
    },
    output: {
        filename: '[name].js',
        path: path.resolve(__dirname, 'dist'),
    },
    module: {
        rules: [
           {
               test: /\.css$/,
               use: ExtractTextWebpackPlugin.extract({
                   fallback: "style-loader",
                   use: [{
                       loader: 'css-loader',
                       options: {
                           modules: true,
                           localIdentName: '[path]_[name]_[local]_[hash:base64:5]'
                       }
                   }]
               })
           }
        ]
     },
plugins: [
+       new HtmlWebpackPlugin({
+           filename: 'main.html',
+           template: './src/template.html',
+           title: 'Webpack App title from webpack'
+       }),
        new CleanWebpackPlugin(['dist']),
        new ExtractTextWebpackPlugin("style.css")
    ]
};
```


__src/template__

``` html
<html>
  <head>
    <meta charset="UTF-8">
    <title><%= htmlWebpackPlugin.options.title %></title>
  <body>
</html>
```
```bash
npm run build  
```
```javascript
//参数说明
new HtmlWebpackPlugin(
    {
        title: 'Webpack App title from webpack', // 用来生成页面的 title 元素
        filename: 'main.html', // 输出的 HTML 文件名，默认是 index.html, 也可以直接配置带有子目录。
        template: './src/template.html' // 模板文件路径，支持加载器，比如 html!./index.html
        inject: 'head', // true | 'head' | 'body' | false  ,注入所有的资源到特定的 template 或者    templateContent 中，如果设置为 true 或者 body，所有的 javascript 资源将被放置到 body 元素的底部，'head' 将放置到 head 元素中。
        favicon: 'path/to/yourfile.ico', // 添加特定的 favicon 路径到输出的 HTML 文件中。
        minify: { // {} | false , 传递 html-minifier 选项给 minify 输出
            removeAttributeQuotes: true // 移除属性的引号
        },
        hash: true, // true | false, 如果为 true, 将添加一个唯一的 webpack 编译 hash 到所有包含的脚本和 CSS件，对于解除 cache 很有用。<script type=text/javascript src=bundle.js?22b9692e22e7be37b57e></script>
        cache: true, // true | false，如果为 true, 这是默认值，仅仅在文件修改之后才会发布文件。
        showErrors:true, // true | false, 如果为 true, 这是默认值，错误信息会写入到 HTML 页面中
        chunks: ['chunk1','chunk2'], // 允许只添加某些块 (比如，仅仅 unit test 块)
        excludeChunks: ['chunk3','chunk4'],// 允许跳过某些块，(比如，跳过单元测试的块)        
    }
)
```
>参考https://segmentfault.com/a/1190000007294861
