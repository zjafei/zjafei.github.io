---
title: 高阶组件：react应用设计模式
date: 2017-11-12 10:22:09
tags: ['pure funtion','纯函数','前端','js','javascript','side effects','副作用','react','组件','高阶组件','Higher Order Components']
category: 'coding'
---

In this article we will discuss how to use Higher Order Components to keep your React applications tidy, well structured and easy to maintain. We’ll discuss how pure functions keep code clean and how these same principles can be applied to React components.
在这片文章里我们将讨论任何使用高阶组件保持你的react应用整洁，结构合理，易于维护。纯函数是如何保持代码整洁并且如何将这些原则应用到react组件中。<!--more-->

## __Pure Functions__ 
## __纯函数__

A function is considered pure if it adheres to the following properties:

如果一个函数遵循以下几个规则那么他就是纯函数：

+ All the data it deals with are declared as arguments
+ 函数处理的数据都是作为参数传递给函数。

+ It does not mutate data it was given or any other data (these are often referred to as side effects).
+ 他不接受不确定的数据或者任何其他数据（他们通常是副作用的）。

+ Given the same input, it will always return the same output.
+ 同样的输入，函数将得到同样的输出。

For example, the add function below is pure:

举个例子，下面的add函数就是个纯函数：

```javascript
function add(x, y) {
    return x + y;
}
```
However, the function badAdd below is impure:

下面badadd就是不是纯函数：
```javascript
var y = 2;

function badAdd(x) {
    return x + y;
} 
```
This function is not pure because it references data that it hasn’t directly been given. As a result, it’s possible to call this function with the same input and get different output:

这个函数之所以不是纯函数是因为他所引用的数据不是直接给予的（不是通过参数传递，而是直接引用了外部全局变量。）。所以当你输入相同的数据时候可能得到不一样的输出。
```javascript
var y = 2;

badAdd(3) // 5

y = 3;

badAdd(3) // 6
```
To read more about pure functions you can read “An introduction to reasonably pure programming” by Mark Brown.

要了解更多关于纯函数的内容，你可以阅读一下Mark Brown写的“An introduction to reasonably pure programming”。

## __Higher Order Functions__ 
## __高阶函数__

A higher order function is a function that when called, returns another function. Often they also take a function as an argument, but this is not required for a function to be considered higher order.

高阶函数的定义是，当一个函数被调用他将输出另一个函数。有时他们把函数作为参数，但是这个不是高阶函数的必要条件。

Let’s say we have our `add` function from above, and we want to write some code so that when we call it we log the result to the `console` before returning the result. We’re unable to edit the `add` function, so instead we can create a new function:

让我们再来看看前面说过的 `add` 函数，我们希望他在返回结果前，`console` 一下结果。我们不能改变 `add` 函数，所以我们要创建一个新函数来替代他：

```javascript
function addAndLog(x, y) {
    var result = add(x, y);
    console.log('Result', result);
    return result;
}
```
We decide that logging results of functions is useful, and now we want to do the same with a subtract function. Rather than duplicate the above, we could write a higher order function that can take a function and return a new function that calls the given function and logs the result before then returning it:

我们显示函数的记录结果是有用的，现在我们要对减法函数做同样的事情。而不是复制上述内容，我们可以编写一个更高阶的函数，它可以执行一个函数接受函数作为参数，被调用的函数会返回结果和显示记录，函数则会返回新的函数。

```javascript
function logAndReturn(func) {
    return function() {
        var args = Array.prototype.slice.call(arguments);
        var result = func.apply(null, args);
        console.log('Result', result);
        return result;
    }
}
```

Now we can take this function `and` use it to add logging to add and `subtract`:

现在我们可以使用这个函数为 `add` 和 `subtract` 函数添加显示记录：

```javascript
var addAndLog = logAndReturn(add);

addAndLog(4, 4) // 8 is returned, ‘Result 8’ is logged

var subtractAndLog = logAndReturn(subtract);

subtractAndLog(4, 3) // 1 is returned, ‘Result 1’ is logged;
```
`logAndReturn` is a HOF because it takes a function as its argument and returns a new function that we can call. These are really useful for wrapping existing functions that you can’t change in behavior. For more information on this, check M. David Green’s article “Higher-Order Functions in JavaScript which goes into much more detail on the subject.

`logAndReturn` 就是一个高阶函数因为他接受一个函数作为参数，并且返回一个函数被我们调用。封装且不改变现有函数的内部实现是非常有用的。希望了解更多相关内容可以去看看M. David Green写的 “Higher-Order Functions in JavaScript”。

Additionally you can check out this CodePen, which shows the above code in action.

另外你可以去CodePen，那里有上面说到的代码。

## __Higher Order Components__ 

## __高阶组件__

Moving into React land, we can use the same logic as above to take existing React components and give them some extra behaviours.

现在我们讨论讨论React，上面讲到的逻辑我们同样可以在React组件上使用，以赋予他们更多的行为。

In this section we’re going to use React Router, the de facto routing solution for React. If you’d like to get started with the library I highly recommend the React Router Tutorial on GitHub.

在这节我们将使用React Router，这个是React的路由解决方案。如果你开始使用库，我强烈建议你使用GitHub上的React Router Tutorial。

## __React Router’s Link component React__

## __路由链接组件__

React Router provides a `<Link>` component that is used to link between pages in a React application. One of the properties that this `<Link>` component takes is `activeClassName`. When a `<Link>` has this property and it is currently active (the user is on a URL that the link points to), the component will be given this class, enabling the developer to style it.

React 路由提供 `<Link>` 用于链接React应用里的页面。`<Link>` 组件其中一个属性是 `activeClassName`。当 `<Link>` 组件处于活动状态时使用了这个属性（用户经过链接时候的意思），组件将使用这个样类名，以便开发人员能够对其样式进行美化。

This is a really useful feature, and in our hypothetical application we decide that we always want to use this property. However, after doing so we quickly discover that this is making all our `<Link>` components very verbose:

这是一个非常有用的功能，在我们假设的应用程序中，我们决定我们总是使用这个属性。然而，很快我们会发现这会使 `<Link>` 组件变得臃肿：

```html
<Link to="/" activeClassName="active-link"> Home </Link>
<Link to="/about" activeClassName="active-link"> About</Link>
<Link to="/contact" activeClassName="active-link"> Contact</Link>
```
Notice that we are having to repeat the class name property every time. Not only does this make our components verbose, it also means that if we decide to change the class name we’ve got to do it in a lot of places.

注意我们每次都必须重复使用那个类的属性名。这不仅使我们的组件臃肿，这也意味着如果我们决定改变类名，我们必须在很多地方进行修改。

Instead, we can write a component that wraps the `<Link>` component:

替换，我们要写一个组件去封装 `<Link>` 组件：

```javascript
var AppLink=React.createClass({
    render:function(){
        return(
            <Link to={this.props.to} activeClassName="active-link">
                {this.props.children}
            </Link>;
        );
    }
});
```

And now we can use this component, which tidies up our links:

现在我们可以使用这个组件来整合我们的链接：

```html
<AppLink to="/">Home</AppLink>
<AppLink to="/about">About</AppLink>
<AppLink to="/contact">Contact</AppLink>
```

In the React ecosystem these components are known as higher order components because they take an existing component and manipulate it slightly without changing the existing component. You can also think of these as wrapper components, but you’ll find them commonly referred to as higher order components in React-based content.

在React生态系统中，这些组件被称为高阶组件，因为他们采用现有组件并对他们进行封装，但是并没有更改现有组件。你也可以将他们视为封装组件，但你会发现他们通常被称为 `React-based` 内容中的高阶组件。

## __Functional, Stateless Components__
## __函数式无状态组件__

React 0.14 introduced support for functional, stateless components. These are components that have the following characteristics:

React0.14开始提供函数式无状态组件，函数式无状态组件具有以下特点：

+ They do not have any state.

+ 他们没有state

+ They do not use any React lifecycle methods (such as componentWillMount()).

+ 他们无法使用React生命周期函数（比如 componentWillMount()）

+ They only define the render method and nothing more.

+ 他们只定义了 render 方法，没有其他的了。

When a component adheres to the above, we can define it as a function, rather than using React.createClass (or class App extends React.Component if you’re using ES2015 classes). For example, the two expressions below both produce the same component:

当一个组件遵守上述内容时，我们可以将其定义为一个函数，而不是使用React.createClass（如果你使用了 ES2015  classes 你也可以使用class 来扩展你的React 组件）。下面的两个表达式产生相同的组件：

```javascript
var App = React.createClass({
    render: function() {
        return <p>My name is { this.props.name }</p>;
    };
});

var App = function(props) {
    return <p>My name is { props.name }</p>;
};
```

In the functional, stateless component instead of referring to this.props we’re instead passed props as an argument. You can read more about this on the React documentation.

在函数式无状态组件中，不是引用this.props，而是将props用作参数。React文档有更多介绍。

Because higher order components often wrap an existing component you’ll often find you can define them as a functional component. For the rest of this article I’ll do that whenever possible.

因为更高阶的组件通常会封装一个现有的组件，所以您经常会发现可以将它们定义为函数组件。在本文的其余部分，我将尽可能地这样做。

## __Better Higher Order Components__
## __更好的高阶组件__

The above component works, but we can do much better. The AppLink component that we created isn’t quite fit for purpose.

我们不只局限于组件可以正常运行，我们希望做的更好。我们创建的 `AppLink` 组件就不太适合。

## __Accepting multiple properties__
## __接受多个参数__

The `<AppLink>` component expects two properties:

`<AppLink>` 组件估计会有两个参数：
+ this.props.to which is the URL the link should take the user to
+ this.props.to 表示链接让用户访问的URL

+ this.props.children which is the text shown to the user
+ this.props.children 表示显示的文本

However, the `<Link>` component accepts many more properties, and there might be a time when you want to pass extra properties along with the two above which we nearly always want to pass. We haven’t made `<AppLink>` very extensible by hard coding the exact properties we need.

当然，`<Link>` 组件可以接受更多发属性，有时我们只是想组件允许传递额外的属性以及上述的两个属性。我们并没有通过代码属性让 `<AppLink>` 有很高的扩展性。

## __The JSX spread__
## __JSX 传递__

JSX, the HTML-like syntax we use to define React elements, supports the spread operator for passing an object to a component as properties. For example, the code samples below achieve the same thing:

JSX，一种类似于HTML的语法用于定义React组件，支持扩展操作符将对象作为属性传递给组件。例如，下面的代码示例实现了同样的事情：

```javascript
var props={a:1,b:2};
<Foo a={props.a }b={props.b} />

<Foo {...props} />
```

Using `{...props}` spreads each key in the object and passes it to `Foo` as an individual property.

使用 `{... props}` 扩展对象中的每个键，并将其作为单个属性传递给 `Foo`。

We can make use of this trick with `<AppLink>` so we support any arbitrary property that `<Link>` supports. By doing this we also future proof ourselves; if `<Link>` adds any new properties in the future our wrapper component will already support them. While we’re at it, I’m also going to change  `AppLink` to be a functional component.

我们可以利用 `<AppLink>` 这个技巧，让我们支持 `<Link>` 支持的任意属性。通这样做，我们也将来证明自己；如果 `<Link>` 在将来添加任何新的属性，我们的包装器组件将已经支持它们。在我们这样做的时候，我同时要将 `AppLink` 改成函数式组件。

```javascript
var AppLink = function(props){
    return <Link {...props} activeClassName="active-link" />;
}
```

Now <Link> will accept any properties and pass them through. Note that we also can use the self closing form instead of explicitly referencing {props.children} in-between the <Link> tags. React allows children to be passed as a regular prop or as child elements of a component between the opening and closing tag.

现在将接受任何属性并传递它们。请注意，我们还可以使用笔和标签，而不是在标签之间传递{props.children}。React允许 children 作为常规 prop 或作为在开始和结束标签之间的组件的子元素进行传递。

## __Property ordering in React__
## __React中属性的排序__

Imagine that for one specific link on your page, you have to use a different activeClassName. You try passing it into <AppLink>, since we pass all properties through:

想象一下，对于页面上的一个特定链接，你必须使用不同的 activeClassName。尝试将其传递到 <AppLink> 中，我们通过以下方式传递所有属性： 

```jsx
<AppLink to=“/special-link” activeClassName=“special-active” >
    Special Secret Link
</AppLink>
```

However, this doesn’t work. The reason is because of the ordering of properties when we render the `<Link>` component:

但是，这不行。渲染 `<Link>` 组件时属性的顺序：

```jsx
return < Link {...props} activeClassName="active-link" />;
```

When you have the same property multiple times in a React component, the last declaration wins. This means that our last `activeClassName="active-link"` declaration will always win, since it’s placed after `{...this.props}`. To fix this we can reorder the properties so we spread this.props last. This means that we set sensible defaults that we’d like to use, but the user can override them if they really need to:

当您在React组件中多次具有相同的属性时，最后一个会被使用。这就意味着放在 `{...this.props}` 后面的 `activeClassName="active-link"` 会被使用。我们可以把 this.props 放在最后来解决这个问题。这意味着如果用户真的需要我们可以设置一些想要合理的默认值，但是这些默认值是可以被覆盖：

```jsx
return <Link activeClassName="active-link" {...props} />;
```

By creating higher order components that wrap existing ones but with additional behavior, we keep our code base clean and defend against future changes by not repeating properties and keeping their values in just one place.

通过创建一个逛街组件可以使现有的组件得到额外的功能，我们尽量保持我们的代码简洁，并且通过不重复的属性来应对未来的变化，同时把他们的值保存在一个地方。

## __Higher Order Component Creators__
## __高阶组件创造者__

Often you’ll have a number of components that you’ll need to wrap in the same behavior. This is very similar to earlier in this article when we wrapped `add` and `subtract` to add logging to them.

通常你需要给很多组件添加同样的方法。这就和我们前面说到的给 `add` 函数和 `subtract` 函数添加显示输出一样。

Let’s imagine in your application you have an object that contains information on the current user who is authenticated on the system. You need some of your React components to be able to access this information, but rather than blindly making it accessible for every component you want to be more strict about which components receive the information.

让我们想象一下，在你的应用程序中，你有一个对象，其中包含关于系统进行身份验证的当前用户的信息。你需要你的 React 组件可以访问这些信息，而不是盲目地使每个组件都可以访问，你更希望严格地控制哪些组件接收信息。

The way to solve this is to create a function that we can call with a React component. The function will then return a new React component that will render the given component but with an extra property which will give it access to the user information.

解决的方法就是我们可以创建一个函数去调用一个 React 组件。函数会返回一个可以访问用户信息并且把原来组件进行渲染的新 React 组件。

That sounds pretty complicated, but it’s made more straightforward with some code:

听起来很复杂，但是通过一些代码就可以很简洁了：

```javascript
function wrapWithUser(Component){
    // information that we don’t want everything to access
    // 收限制的用户信息
    var secretUserInfo = {
        name:'Jack Franklin',
        favouriteColour:'blue'
    };

    // return a newly generated React component
    // using a functional, stateless component
    // 返回一个新的 React 组件
    // 使用一个无状态函数式组件
    return function(props){
        // pass in the user variable as a property, along with
        // all the other props that we might be given
        // 用户信息作为属性进行传递
        // 其他属性我们可以自定义
        return <Component user={secretUserInfo} {...props} />
    }
}
```

The function takes a React component (which is easy to spot given React components have to have capital letters at the beginning) and returns a new function that will render the component it was given with an extra property of user, which is set to the `secretUserInfo`.

函数通过一个React组件（自定义的React组件必须有大写字母开头）并返回一个新的函数，它将使用设置在 `secretUserInfo` 里的用户的属性来渲染那个组件。

Now let’s take a component, `<AppHeader>`, which wants access to this information so it can display the logged in user:

现在我们有一个 `<AppHeader>` 组件需要这些信息，以便把登陆的用户信息展示出来：

```javascript
var AppHeader=function(props){
    if(props.user){
        return <p>Logged in as {props.user.name} </p>;
    } else {
        return <p>You need to login</p>;
    }
}
```

The final step is to connect this component up so it is given `this.props.user`. We can create a new component by passing this one into our `wrapWithUser` function.

最后一步是将此组件连接起来，我们可以通过 `wrapWithUser` 函数创建新的组件，以便把 `this.props.user` 传递给他。

```javascript
var ConnectedAppHeader = wrapWithUser(AppHeader);
```

We now have a `<ConnectedAppHeader>` component that can be rendered, and will have access to the `user` object.

现在我们有了 `<ConnectedAppHeader>` 组件可以访问 `user` 对象，并且渲染原来的组件。

I chose to call the component `ConnectedAppHeader` because I think of it as being connected with some extra piece of data that not every component is given access to.

我给这个组件起名叫 `ConnectedAppHeader` 因为他传递了一些不希望所有组件都可以使用的额外数据。

This pattern is very common in __React__ libraries, particularly in __Redux__, so being aware of how it works and the reasons it’s being used will help you as your application grows and you rely on other third party libraries that use this approach.

这种模式在 __React__ 库中非常常见，特别是在 __Redux__ 中，知道你依赖的第三方库是如何工作的，它被使用的原因将帮助你随着你的应用程序一起成长。

## __Conclusion__
## __结论__

This article has shown how, by applying principles of functional programming such as pure functions and higher order components to React, you can create a codebase that’s easier to maintain and work with on a daily basis.

这篇文章即将结束，通过应用诸如纯函数和更高阶函数的函数式编程的原理应用到React，您可以创建一个更容易维护和使用的代码库。

By creating higher order components you’re able to keep data defined in only one place, making refactoring easier. Higher order function creators enable you to keep most data private and only expose pieces of data to the components that really need it. By doing this you make it obvious which components are using which bits of data, and as your application grows you’ll find this beneficial.

通过创建更高阶的组件，你只能将数据定义在一个位置，使重构更容易。更高阶的函数创建者使你能够将大部分数据保留为私有，并且仅将数据片段暴露给真正需要的组件。通过这样做，你可以清楚地知道哪些组件正在使用哪些数据，并且随着应用程序的发展，你会发现这是有益的。