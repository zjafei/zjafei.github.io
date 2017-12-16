---
title: react基于state和props的父子组件和兄弟组件沟通
date: 2017-12-16 14:05:41
tags: ['react','state','props','组件沟通','信息传递']
category: 'coding'
---

# react-state react状态管理

React是Facebook开源的一个用于构建用户界面的Javascript库，React专注于MVC架构中的V，即视图。

## Virtual DOM

是一个模拟 DOM 树的 JavaScript 对象。 React 使用 Virtual DOM 来渲染 UI， 同时监听 Virtual DOM 上数据的变化并自动迁移这些变化到 UI 上。

## React Elements

代表 HTML 元素的 JavaScript 对象。 这些 JavaScript 对象最后被编译成对应的 HTML 元素。

## JSX

React 定义的一种 JavaScript 语法扩展，类似于 XML 。 JSX 是可选的， 我们完全可以使用 JavaScript 来编写 React 应用， 不过 JSX 提供了一套更为简便的方式来写 React 应用。<!--more-->

```JSX
var div = React.createElement('div', null, "Hello React");
// 使用 JSX
var div = <div>Hello React</div>;
```

## Components

封装 React Elements， 包含一个或者多个 React Elements。 Components 用于创建可以复用的 UI 模块，例如 分页，侧栏导航等。

```JSX
var HelloMessage = React.createClass({
  render() {
    return <div>Hello {this.props.name}</div>;
  }
});
```

## Props

在上一个例子中，可以看到有一个特殊的引用： `this.props.name`。 这个引用称之为 `Props`，可以将他想象成 Component 的设置选项。

在使用上， Props 类似于 HTML 中的属性：

```JSX
var HelloMessage = React.createClass({
  render() {
    return <HelloMessage name="foo" />;
  }
});
```

在 Component 内部，通过 this.props.name 来引用这个 Props：

```JSX
var HelloMessage = React.createClass({
  render() {
    return <div>Hello {this.props.name}</div>;
  }
});
```

需要注意的是， Props 仅用来定制 Component， 这些数据不应该被改动。 如果涉及到需要做改动的数据， 得考虑使用 `state`。

## Stateful Component
State 数据代表 Component 的状态， 用于维护 Component 内部的状态。 当 State 发生改变之后， React 将会重新渲染 UI 。State 数据通过 this.state 访问：

```JSX
var Greeting = React.createClass({
  constructor () {
    this.state = { greeted: false };
  },
  render() {
    var response = this.state.greeted ? 'Yes' : 'No';

    return (
      <div>
        <div>Hello React</div>
        <span>{response}</span>
        <button onClick={()=>{
            this.setState({
                    greeted: !this.state.greeted
                });
            },}>
            Hi
        </button>
      </div>
    );
  }
});
```
![dom action state的关系图](/0/state.png)

当 State 发生改变后， React 将 Component 渲染到 Virtual DOM，新的 Virtual DOM 与 旧版本的进行比对，检查出改变的部分并更新浏览器的 DOM。 在这个例子中，当按钮被点击后， `greeted`状态数据发生了变化，UI 跟随着更新。


## Props与State的结合
```JSX
var Greeting = React.createClass({
  constructor () {
    this.state = { greeted: false };
  },
  
  render() {
    var response = this.state.greeted ? 'Yes' : 'No';

    return (
      <div>
        <div>Hello {this.props.name}</div>
        <span>{response}</span>
         <button onClick={()=>{
            this.setState({
                    greeted: !this.state.greeted
                });
            },}>
            Hi
        </button>
      </div>
    );
  }
});

var users = ["foo", "bar", "baz"];

var GreetingApp = React.createClass({
  render() {
    var greetings = this.props.users.map((user)=>{
      return <Greeting name={user} />;
    });

    return <div>{greetings}</div>;
  }
});
```

## 单向数据流

React是单向数据流，数据主要从父节点传递到子节点（通过props）。

如果父级的某个props改变了，React会重渲染所有的子节点。

### 父子组件沟通
* 父组件更新组件状态  -----props----->　子组件更新
* 子组件更新父组件状态   -----需要父组件传递回调函数----->  子组件调用触发

>`子组件更新父组件就需要 父组件通过props传递一个回调函数到子组件中，这个回调函数可以更新父组件，子组件就是通过触发这个回调函数，从而使父组件得到更新。`

![父子组件沟通](/0/propsFlow.png)

```JSX
const Greeting = React.createClass({
  constructor () {
    this.state = { greeted: false };
  },
  render() {
    var response = this.state.greeted ? 'Yes' : 'No';

    return (
      <div>
        <div>Hello {this.props.name}</div>
        <span>{response}</span>
        <button onClick={()=>{
            const greeted = !this.state.greeted;
            this.setState({
                    greeted: greeted,
                });
            this.props.changeName(greeted?'eric':'mazheng');   
            },}>
            Hi
        </button>
      </div>
    );
  }
});

const UseGreeting = React.createClass({
  constructor () {
    this.state = { name: 'eric' };
  },
  render() {
    return (
        <Greeting
            name={this.state.name}
            changeName={(newName)=>{
                this.setState({
                    name: newName,
                }); 
            }}
        />
    );
  }
});
```

### 兄弟组件沟通

* 按照React单向数据流方式，我们需要借助父组件进行传递，通过父组件回调函数改变兄弟组件的props。

![兄弟组件沟通](/0/brothers.png)

