---
title: react新生命周期替代旧生命周期的最佳实践
date: 2019-02-20 08:17:39
tags: ['react','生命周期','Fiber','getSnapshotBeforeUpdate','getDerivedStateFromProps','最佳实践']
category: 'coding'
---
## 1. Fiber

很长时间React团队一直致力于实现异步渲染以减少交互的卡顿现象，最后的产物就是Fiber。

Fiber的基本原理：

1.把可中断的工作拆分成小任务

2.对正在做的工作调整优先次序、重做、复用上次（做了一半的）成果

3.在父子任务之间从容切换（yield back and forth），以支持React执行过程中的布局刷新

总结：改变原来一次性渲染到结束的不可控的线性栈式数据结构，变为多次渲染可控的小颗粒状的链式数据结构。做到主要的先渲染，耗时渲染可中断，未完成的可继续。

<!--more-->

## 2. 导致问题
由于渲染的过程是可以中断的异步行为，所以`componentWillMount`、`componentWillReceiveProps`、`componentWillUpdate`会在一次渲染过程中被重复调用，而导致一些问题。react17版本将弃用这些生命周期函数，但是可以使用带`UNSAFE_`的前缀的生命周期函数来替代使用。

## 3. 从传统生命周期迁移
为了解决上面的问题react引入了两个新的生命周期，`static getDerivedStateFromProps`和`getSnapshotBeforeUpdate`。

### getDerivedStateFromProps

组件实例化以及重新渲染之前会调用这个函数，输出一个对象去更新 state ，或者输出 null 表明组件不需要任何状态更新。与 componentDidUpdate 一起使用，将代替 componentWillReceiveProps。

```javascript
class Example extends React.Component {
  static getDerivedStateFromProps(props, state) {
    // ...
  }
}
```

### getSnapshotBeforeUpdate

进行变化（例如在更新DOM之前）之前会调用这个函数。此生命周期的返回值将作为第三个参数传递给componentDidUpdate。与 componentDidUpdate 一起使用，将代替 componentWillUpdate。

```javascript
class Example extends React.Component {
  getSnapshotBeforeUpdate(prevProps, prevState) {
    // ...
  }
}
```

## 4. 事例

### 1. 初始化状态

```javascript
// 旧代码
class ExampleComponent extends React.Component {
  state = {};

  componentWillMount() {
    this.setState({
      currentColor: this.props.defaultColor,
      palette: 'rgb',
    });
  }
}
```

```javascript
// 新代码
class ExampleComponent extends React.Component {
  state = {
    currentColor: this.props.defaultColor,
    palette: 'rgb',
  };
}
```

**总结：状态的初始化直接在实例化的时候进行。**

### 2. 远端数据获取

```javascript
// 旧代码
class ExampleComponent extends React.Component {
  state = {
    externalData: null,
  };

  componentWillMount() {
    this._asyncRequest = loadMyAsyncData().then(
      externalData => {
        this._asyncRequest = null;
        this.setState({externalData});
      }
    );
  }

  componentWillUnmount() {
    if (this._asyncRequest) {
      this._asyncRequest.cancel();
    }
  }

  render() {
    if (this.state.externalData === null) {
      // Render loading state ...
    } else {
      // Render real UI ...
    }
  }
}
```

```javascript
// 新代码
class ExampleComponent extends React.Component {
  state = {
    externalData: null,
  };

  componentDidMount() {
    this._asyncRequest = loadMyAsyncData().then(
      externalData => {
        this._asyncRequest = null;
        this.setState({externalData});
      }
    );
  }

  componentWillUnmount() {
    if (this._asyncRequest) {
      this._asyncRequest.cancel();
    }
  }

  render() {
    if (this.state.externalData === null) {
      // Render loading state ...
    } else {
      // Render real UI ...
    }
  }
}
```

**总结：componentDidMount中进行数据的请求。**

### 3. 添加事件侦听器（或订阅）

```javascript
// 旧代码
class ExampleComponent extends React.Component {
  componentWillMount() {
    this.setState({
      subscribedValue: this.props.dataSource.value,
    });

    // 不安全，会导致泄漏
    this.props.dataSource.subscribe(
      this.handleSubscriptionChange
    );
  }

  componentWillUnmount() {
    this.props.dataSource.unsubscribe(
      this.handleSubscriptionChange
    );
  }

  handleSubscriptionChange = dataSource => {
    this.setState({
      subscribedValue: dataSource.value,
    });
  };
}
```

```javascript
// 新代码
class ExampleComponent extends React.Component {
  state = {
    subscribedValue: this.props.dataSource.value,
  };

  componentDidMount() {
    //事件监听器只能在mount之后添加，
    //因此如果挂载中断或错误，它们不会泄漏。   
    this.props.dataSource.subscribe(
      this.handleSubscriptionChange
    );

    //外部值可能会在渲染和装载之间发生变化
    //在某些情况下，处理这种情况可能很重要。
    if (
      this.state.subscribedValue !==
      this.props.dataSource.value
    ) {
      this.setState({
        subscribedValue: this.props.dataSource.value,
      });
    }
  }

  componentWillUnmount() {
    this.props.dataSource.unsubscribe(
      this.handleSubscriptionChange
    );
  }

  handleSubscriptionChange = dataSource => {
    this.setState({
      subscribedValue: dataSource.value,
    });
  };
}
```

**总结：componentDidMount中进行数据的绑定。**

### 4. 基于 props 更新 state

```javascript
// 旧代码
class ExampleComponent extends React.Component {
  state = {
    isScrollingDown: false,
  };

  componentWillReceiveProps(nextProps) {
    if (this.props.currentRow !== nextProps.currentRow) {
      this.setState({
        isScrollingDown:
          nextProps.currentRow > this.props.currentRow,
      });
    }
  }
}
```

```javascript
// 新代码
class ExampleComponent extends React.Component {
  //在构造函数中初始化状态，
  //或者使用属性初始值设定项。
  state = {
    isScrollingDown: false,
    lastRow: null,
  };

  static getDerivedStateFromProps(props, state) {
    if (props.currentRow !== state.lastRow) {
      return {
        isScrollingDown: props.currentRow > state.lastRow,
        lastRow: props.currentRow,
      };
    }

    // 返回null表示状态没有变化。
    return null;
  }
}
```

**总结：使用 getDerivedStateFromProps 代替 componentWillReceiveProps。用 state 的属性代替 nextProps**

### 5. 调用外部回调

```javascript
// 旧代码
class ExampleComponent extends React.Component {
  componentWillUpdate(nextProps, nextState) {
    if (
      this.state.someStatefulValue !==
      nextState.someStatefulValue
    ) {
      nextProps.onChange(nextState.someStatefulValue);
    }
  }
}
```

```javascript
// 新代码
class ExampleComponent extends React.Component {
  componentDidUpdate(prevProps, prevState) {
    if (
      this.state.someStatefulValue !==
      prevState.someStatefulValue
    ) {
      this.props.onChange(this.state.someStatefulValue);
    }
  }
}
```

**总结：用 componentDidUpdate 代替 componentWillUpdate。**


### 6. 有副作用的props更新

```javascript
// 旧代码
class ExampleComponent extends React.Component {
  componentWillReceiveProps(nextProps) {
    if (this.props.isVisible !== nextProps.isVisible) {
      logVisibleChange(nextProps.isVisible);
    }
  }
}
```

```javascript
// 新代码
class ExampleComponent extends React.Component {
  componentDidUpdate(prevProps, prevState) {
    if (this.props.isVisible !== prevProps.isVisible) {
      logVisibleChange(this.props.isVisible);
    }
  }
}
```

**总结：使用 componentDidUpdate 代替 componentWillReceiveProps。**

### 7. props 改变后请求远端数据

```javascript
// 旧代码
class ExampleComponent extends React.Component {
  state = {
    externalData: null,
  };

  componentDidMount() {
    this._loadAsyncData(this.props.id);
  }

  componentWillReceiveProps(nextProps) {
    if (nextProps.id !== this.props.id) {
      this.setState({externalData: null});
      this._loadAsyncData(nextProps.id);
    }
  }

  componentWillUnmount() {
    if (this._asyncRequest) {
      this._asyncRequest.cancel();
    }
  }

  render() {
    if (this.state.externalData === null) {
      // Render loading state ...
    } else {
      // Render real UI ...
    }
  }

  _loadAsyncData(id) {
    this._asyncRequest = loadMyAsyncData(id).then(
      externalData => {
        this._asyncRequest = null;
        this.setState({externalData});
      }
    );
  }
}
```

```javascript
// 新代码
class ExampleComponent extends React.Component {
  state = {
    externalData: null,
  };

  static getDerivedStateFromProps(props, state) {
    //存储prevId状态，以便我们可以比较道具变化的时间。
    //清除以前加载的数据（因此我们不会渲染陈旧的东西）。
    if (props.id !== state.prevId) {
      return {
        externalData: null,
        // 代替 prevProps.id
        prevId: props.id,
      };
    }

    // 状态没有改变
    return null;
  }

  componentDidMount() {
    this._loadAsyncData(this.props.id);
  }

  componentDidUpdate(prevProps, prevState) {
    if (this.state.externalData === null) {
      this._loadAsyncData(this.props.id);
    }
  }

  componentWillUnmount() {
    if (this._asyncRequest) {
      this._asyncRequest.cancel();
    }
  }

  render() {
    if (this.state.externalData === null) {
      // 渲染加载状态...
    } else {
      // 渲染UI ...
    }
  }

  _loadAsyncData(id) {
    this._asyncRequest = loadMyAsyncData(id).then(
      externalData => {
        this._asyncRequest = null;
        this.setState({externalData});
      }
    );
  }
}
```
**总结：使用 componentDidUpdate 代替 componentWillReceiveProps，业务的关键状态用state临时保存**

### 7. 在更新之前读取DOM属性

```javascript
// 旧代码
class ScrollingList extends React.Component {
  listRef = null;
  previousScrollOffset = null;

  componentWillUpdate(nextProps, nextState) {
    //我们是否在列表中添加新元素？
    //捕获滚动位置，以便我们稍后调整滚动。
    if (this.props.list.length < nextProps.list.length) {
      this.previousScrollOffset =
        this.listRef.scrollHeight - this.listRef.scrollTop;
    }
  }

  componentDidUpdate(prevProps, prevState) {
    //如果设置了previousScrollOffset，我们刚刚添加了新项目。
    //调整滚动，以便这些新项目不会将旧项目推出视图。
    if (this.previousScrollOffset !== null) {
      this.listRef.scrollTop =
        this.listRef.scrollHeight -
        this.previousScrollOffset;
      this.previousScrollOffset = null;
    }
  }

  render() {
    return (
      <div ref={this.setListRef}>
        {/* ...内容... */}
      </div>
    );
  }

  setListRef = ref => {
    this.listRef = ref;
  };
}
```

```javascript
// 新代码
class ScrollingList extends React.Component {
  listRef = null;

  getSnapshotBeforeUpdate(prevProps, prevState) {
    //我们是否在列表中添加新项目？
    //捕获滚动位置，以便我们稍后调整滚动。
    if (prevProps.list.length < this.props.list.length) {
      return (
        this.listRef.scrollHeight - this.listRef.scrollTop
      );
    }
    return null;
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    //如果我们有快照值，我们刚刚添加了新项目。
    //调整滚动，以便这些新项目不会将旧项目推出视图。
    //这里的 snapshot 就是 getSnapshotBeforeUpdate 的返回
    if (snapshot !== null) {
      this.listRef.scrollTop =
        this.listRef.scrollHeight - snapshot;
    }
  }

  render() {
    return (
      <div ref={this.setListRef}>
        {/* ...内容... */}
      </div>
    );
  }

  setListRef = ref => {
    this.listRef = ref;
  };
}
```

**总结：getSnapshotBeforeUpdate 代替 componentDidUpdate**

## 4. 写在最后的总结

### 1. 原来几个含有will的生命周期函数基本上是没有用了，就算用也是用在不触发UI改变的业务上。

### 2. 现在能比较前后状态的生命周期函数只有 componentDidUpdate 和 getSnapshotBeforeUpdate。

### 3. 比较前后状态的能用 componentDidUpdate 解就用 componentDidUpdate 解决。不能满足的用 static getDerivedStateFromProps 来代替。

### 4. 使用 static getDerivedStateFromProps 的要把关键的业务状态添加到 state 里作为 prevProps 的替代