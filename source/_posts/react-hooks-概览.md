---
title: react hooks（钩子）概览和一些实践
date: 2019-02-22 20:13:15
tags: ['react','hooks','useState','useEffect','自定义钩子']
category: 'coding'
---
>钩子是 React 16.8中的新增功能。它们允许您在不编写类的情况下使用 state 和其他 React 功能。

钩子是向下兼容的。在我介绍向你介绍这个新功能前你最好对 React 有所了解。

<!--more-->

## 状态钩子

下面是个计数器的事例。当你点击按钮时，数值就会增加：

```JSX
import React, { useState } from 'react';

function Example() {
  // 声明一个新的状态变量，我们将其称为“count”
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

在这里 `useState` 就是一个钩子（过会我们再说这样做的意义）。在函数组件里面使用钩子为函数组件添加内部状态。React 将在每次渲染之间保留此状态。`useState` 方法会返回一个长度为2的数组。第一个元素就是你定义的状态，第二个元素就是用来更新这个状态的方法。你可以在处理事件的时候来调取这个方法。这个和类里面的 `this.setState` 和像，但是他不能像 `this.setState` 那样同时合并新旧状态。

`useState` 接受一个参数作为状态的初始值。在刚刚的事例中，这个值就是 `0`，因为通常计算都是从零开始的。但是不同于 `this.state`，钩子的状态不一定必须是对象。初始状态参数只在第一次渲染的时使用。

## 声明多个状态变量

你可以在同一个组件里面定义多个状态钩子

```JSX
function ExampleWithManyStates() {
  // 声明多个状态变量！
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
  // ...
}
```

数组解构语法允许我们为通过调用 `useState` 声明的状态变量赋予不同的名称。命名并不是 `useState` API的一部分。React 假设如果多次调用 `useState`，则在每次渲染期间以相同的顺序执行。下面我们将讨论为什么要这样做，以及何时使用他。

## 什么是钩子？

钩子函数可以使你的函数组件拥有 React state 和生命周期功能。钩子在类内部不起作用----它们允许你在没有类的情况下使用 React。（我们并不建议你重写你现有的组件，但是你可以在新的组件里面使用钩子。）

React 提供了一些像 `useState` 这样的内置钩子。你还可以创建自己的钩子以复用不同组件之间的状态行为。我们先来看看内置的钩子。

## 作用钩子

在 React 组件修改 DOM 前你一般都需要获取数据。我们管这种操作叫 `“side effects”（副作用）`（有时也简称“effects”），因为他们功能同时影响其他组件并且渲染无法完成。

副作用钩子 `useEffect` 为组件添加了执行副作用操作的功能。这个与 React 类中的 `componentDidMount`，`componentDidUpdate` 和 `componentWillUnmount` 具有相同的用途，但是统一成为了一个API。（过会我们将展示一个使用 `useEffect` 副作用钩子的函数。 ）

例如，此组件在 React 更新 DOM 后设置文档标题：

```JSX
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // 类似 componentDidMount 和 componentDidUpdate:
  useEffect(() => {
    // 通过浏览器的 API 更新文档的标题
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

但你调用 `useEffect` 的时候，你就在告诉 React 在重新渲染 DOM 之后执行你的作用函数。副作用函数是在组件内声明，因此可以访问其 props 和 state。默认情况下 React 在每次渲染后都会执行作用函数----同样包括第一次渲染。（我们将更多地讨论如何使用效果钩，以及他与类的生命周期的差异。）

作用函数还可以根据返回函数来指定如何“清理”它们。例如，下面这个组件使用作用函数来订阅朋友的在线状态，并通过取消订阅来清理状态：

```JSX
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);

    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

在这个事例中， React 会在组件销毁的时候通过 `ChatAPI` 来取消订阅。而且每次渲染的时候都会重新执行效果函数。（甚至当传递给 `ChatAPI` 的  `props.friend.id` 没有改变的情况下可以告诉 React 忽略重新订阅。）

就像使用 `useState` 一样，您可以在组件中使用多个作用函数：

```JSX
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }
  // ...
```

通过钩子，您可以根据需要组织组件中的副作用函数（例如添加和删除订阅），而不是强制通过生命周期的函数进行处理。

## 钩子的规则

钩子本质就是一个 JavaScript 函数，但是他有两个特殊的规则：

* 只能在 **顶层** 调用钩子。不要在循环，条件或嵌套函数中调用钩子。

* 只能在 React 函数组件中调用钩子。不要在一般的 JavaScript 函数里调用钩子。（还有一个可以调用钩子的地方----自定义钩子。马上我们就会了解他。）

## 自定义钩子

有时，我们希望在组件之间复用一些有状态逻辑。一般有两个方法解决这个问题：`高价组件` 和 `渲染参数（render  props）`。自定义钩子可以在不添加组件的情况下解决上面的问题。

前面我们介绍了一个通过 `useState` 和 `useEffect` 钩子来订阅朋友在线状态的组件。现在我们想把这个订阅的逻辑使用到其他组件中去。

首先我们将创建自定义钩子组件 `useFriendStatus` 用来封装这个逻辑。

```JSX
import React, { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

使用 `friendID` 作为参数，并返回我们的朋友是否在线。

现在我们可以在其他组件中使用它：

```JSX
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

```JSX
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

这些组件的状态完全独立。钩子只是复用 *状态逻辑*` 的一种方式，而不是状态本身。事实上，每次调用钩子都有一个完全孤立的状态----所以你甚至可以在一个组件中使用两次相同的自定义钩子。

您可以编写自定义钩子，涵盖各种用例，如表单处理，动画，声明订阅，计时器，以及可能还有更多我们想到的用例。我们希望自定义钩子可以在 React 社区火起来。

## 其他钩子

你可能会发现一些不太常用的内置钩子也很有用。例如，`useContext` 允许你在不引入嵌套的情况下订阅 React 上下文：

```JSX
function Example() {
  const locale = useContext(LocaleContext);
  const theme = useContext(ThemeContext);
  // ...
}
```

`useReducer` 允许你使用 reducer 管理复杂组件的本地状态：

```JSX
function Todos() {
  const [todos, dispatch] = useReducer(todosReducer);
  // ...
```

## 推荐阅读
* [添加钩子功能的初衷](https://reactjs.org/docs/hooks-intro.html)
* [状态钩子](https://reactjs.org/docs/hooks-state.html)
* [钩子API介绍](https://reactjs.org/docs/hooks-reference.html)
* [钩子常见问题](https://reactjs.org/docs/hooks-faq.html)

