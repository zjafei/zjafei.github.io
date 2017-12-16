---
title: react通过redux和redux-saga管理状态
date: 2017-12-16 19:53:12
tags: ['reduxe','redux-saga','effect','dispatch','reducer']
---
## Redux
### 应用场景
* 某个组件的状态，需要共享
* 某个状态需要在任何地方都可以拿到
* 一个组件需要改变全局状态
* 一个组件需要改变另一个组件的状态<!--more-->
### 设计思想
* Web 应用是一个状态机，视图与状态是一一对应的。(state)
* 所有的状态，保存在一个对象里面。（store）

![reduxe组件沟通](/0/reducer.png)

组件的动作如果dispatch去调用全局纯函数reducer，reducer来修改全局对象的store里的state，组件重新渲染。

解决了跨组件的状态交互。
## Redux-saga
### effect副作用函数（异步函数）
![effect](/0/effect.png)

* Generator 函数的语法 

```javascript
function* helloWorldGenerator() {
  yield 'hello';
  yield 'world';
  return 'ending';
}

var hw = helloWorldGenerator();

hw.next()
// { value: 'hello', done: false }

hw.next()
// { value: 'world', done: false }

hw.next()
// { value: 'ending', done: true }

hw.next()
// { value: undefined, done: true }
```

>callback --> Promise then 连死调用 --> Generator yield 函数同步化
