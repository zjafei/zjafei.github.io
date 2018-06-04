---
title: 基于nodejs的简单RESTful服务代码
date: 2018-06-04 16:56:20
tags: ['nodejs','RESTful','http','cURL']
category: 'coding'
---
代码很见简单主要是用来练习nodejs的http模块的，随便了解了解RESTful和cURL命令。代码走起……
<!--more-->
```javascript
const http = require('http');
const url = require('url');

const items = [];

const server = http.createServer(function (req, res) {
    const path = url.parse(req.url).pathname;
    const index = parseInt(path.slice(1), 10);
    switch (req.method) {
        case 'POST':
            let item = '';
            req.setEncoding('utf8');// 数据设为字符串
            req.on('data', function (chunk) {// 读入新的数据块就会触发 data 事件 默认是 buffer 字节数组
                item = item + chunk
            });
            req.on('end', function () {// 数据读完了触发 end 事件
                items.push(item);
                res.end('OK\n');
            });
            break;
        case 'GET':
            let body = items.map((item, index) => {
                return index + '\t' + item;
            }).join('\n') + '\n';
            res.setHeader('Content-Length', Buffer.byteLength(body));
            res.setHeader('Content-Type', 'text/plain; charset="utf8"');
            res.end(body);
            break;
        case 'DELETE':
            if (isNaN(index)) {
                res.statusCode = 400;// 表示无效
                res.end('Invalid item id');
            } else if (!items[index]) {
                res.statusCode = 404;// 表示不存在
                res.end('item not found');
            } else {
                items.splice(index, 1);
                res.end('OK\n');
            }
            break;
        case 'PUT':
            if (isNaN(index)) {
                res.statusCode = 400;// 表示无效
                res.end('Invalid item id');
            } else if (!items[index]) {
                res.statusCode = 404;// 表示不存在
                res.end('item not found');
            } else {
                let item = '';
                req.setEncoding('utf8');// 数据设为字符串
                req.on('data', function (chunk) {// 读入新的数据块就会触发 data 事件 默认是 buffer 字节数组
                    item = item + chunk
                });
                req.on('end', function () {// 数据读完了触发 end 事件
                    items[index] = item;
                    res.end('OK\n');
                });
                res.end('OK\n');
            }
            break;
        default:
            break;
    }
});
server.listen(3000);
console.log('server is running');
```

## 下面说说`RESTful`：
> `表现层状态转换 (REST)`是一种基于HTTP定义一组约束和属性的架构风格。符合REST架构风格或REST风格的Web服务的Web服务提供了互联网上计算机系统之间的交互操作。符合REST的Web服务允许请求系统通过使用统一和预定义的一组无状态操作来访问和操作文本表示的Web资源。

wiki百科 [`Representational state transfer`](https://en.wikipedia.org/wiki/Representational_state_transfer)

## 下面说说`cURL`：
 > `cURL`是利用URL语法在命令行方式下工作的开源文件传输工具。它被广泛应用在Unix、多种Linux发行版中，并且有DOS和Win32、Win64下的移植版本。

wiki百科 [`cURL`](https://en.wikipedia.org/wiki/CURL)

### 基本语法
```bash
# GET
curl http://xxxx.xxx
# POST
curl -d 'post something' http://xxxx.xxx
# DELETE
curl -XDELETE http://xxxx.xxx/x
# PUT
curl -XPUT -d 'put something' http://xxxx.xxx/x
```
