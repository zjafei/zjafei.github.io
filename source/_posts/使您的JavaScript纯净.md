---
title: 使您的JavaScript纯净
date: 2017-11-11 11:06:59
tags: ['pure funtion','纯函数','前端','js','javascript','side effects','副作用']
category: 'coding'
---

Once your website or application goes past a small number of lines, it will inevitably contain bugs of some sort.

一旦您的网站或应用程序达到一定的代码量，它将不可避免地包含某些类型的错误。

This isn’t specific to JavaScript but is shared by nearly all languages—it’s very tricky, if not impossible, to thoroughly rule out the chance of any bugs in your application.

这不是JavaScript特有的，而是几乎所有的程序语言都面临这个问题 - 这个问题非常棘手，是不是有可能，彻底的排除掉你的程序中出现任何错误的机会。

However, that doesn’t mean we can’t take precautions by coding in a way that lessens our vulnerability to bugs.

然而，这并不意味着我们不能通过编码以某种方式采取预防措施，使我们在bugs面前不显得那么脆弱。<!--more-->

## __Pure and impure functions__
## __纯和不纯的函数__

A pure function is defined as one that doesn’t depend on or modify variables outside of its scope. That’s a bit of a mouthful, so let’s dive into some code for a more practical example.

纯函数定义为不依赖于或修改其范围之外的变量的函数。但看这个定义等于什么也没说，所以让我们来看看一些更实际的例子。

```javascript
function mouseOnLeftSide(mouseX){
    return mouseX < window.innerWidth / 2;
}

document.onmousemove =function(e) {
    console.log(mouseOnLeftSide(e.pageX));
};
```

`mouseOnLeftSide()` takes an X coordinate and checks to see if it’s less than half the window width—which would place it on the left side. However, `mouseOnLeftSide()` is not a pure function. We know this because within the body of the function, it refers to a value that it wasn’t explicitly given:

`mouseOnLeftSide()`采取X坐标并检查是否小于窗口宽度的一半----以便将其放在左侧。但是，`mouseOnLeftSide()`不是纯函数。我们知道这一点，因为在函数体内，参考值不是一个确定值：

```javascript
return mouseX < window.innerWidth / 2;
```

The function is given `mouseX`, but not `window.innerWidth`. This means the function is reaching out to access data it wasn’t given, and hence it’s not pure.

该函数给出了 `mouseX`，但是 `window.innerWidth` 却不明确。这意味着该函数正在访问不确定的数据，因此它是不纯的。

## __The problem with impure functions__
## __不纯的函数是个问题__

You might ask why this is an issue—this piece of code works just fine and does the job expected of it. Imagine that you get a bug report from a user that when the window is less than 500 pixels wide the function is incorrect. How do you test this? You’ve got two options:

你也许会问这怎么就是个问题了，代码不就是找到并且做他要做的事就可以了么。你想象一下，当你从某个用户那里得知当窗口小于500像素的时候功能就不正常了。你怎么测试代码？您已经有两个选项：

1.You could manually test by loading up your browser and moving your mouse around until you’ve found the problem.

1.您可以手动测试，通过浏览器加载你的程序，并且移动鼠标。直到你发现的问题。

2.You could write some unit tests (Rebecca Murphey’s Writing Testable JavaScript is a great introduction) to not only track down the bug, but also ensure that it doesn’t happen again.

2.你可以写一些单元测试（Rebecca Murphey写的 可测试性的JavaScript 是一个很好的介绍）不仅跟踪的错误，也确保错误不再发生。

You could write some unit tests (Rebecca Murphey’s Writing `Testable JavaScript` is a great introduction) to not only track down the bug, but also ensure that it doesn’t happen again.

你可以写一些单元测试（Rebecca Murphey写的 `可测试性的JavaScript` 是一个很好的介绍）不仅跟踪的错误，也确保错误不再发生。

Keen to have a test in place to avoid this bug recurring, we pick the second option and get writing. Now we face a new problem, though: how do we set up our test correctly? We know we need to set up our test with the window width set to less than 500 pixels, but how? The function relies on `window.innerWidth`, and making sure that’s at a particular value is going to be a pain.

热衷有一个测试中的地方，以避免此错误重复，我们选择第二个选项，现在就去写写。现在我们面对了新的问题，虽然：我们怎么才能确保我们的测试是正确的？我们知道这个测试需要一个窗口，而且窗口的宽度要小于500像素。但是怎么做呢？那个函数依赖 `window.innerWidth`，确保他是一个特定的值将会很头疼的。

## __Benefits of pure functions__
## __纯函数的好处__

#### __SIMPLER TESTING__
#### __简单的测试__

With that issue of how to test in mind, imagine we’d instead written the code like so:

记住怎么去测试这个问题，想象一下我们并没有编写下面那样的代码：

```javascript
function mouseOnLeftSide(mouseX, windowWidth) {
    return mouseX < windowWidth / 2;
}

document.onmousemove = function(e) {
    console.log(mouseOnLeftSide(e.pageX, window.innerWidth));
};
```

The key difference here is that `mouseOnLeftSide()` now takes two arguments: the mouse X position and the window width. This means that `mouseOnLeftSide()` is now a pure function; all the data it needs it is explicitly given as inputs and it never has to reach out to access any data.

关键不同点 `mouseOnLeftSide()` 函数有两个参数：鼠标X轴的坐标和窗口的宽度。这就表示 `mouseOnLeftSide()` 现在是个纯函数；函数需要的全部数据都能够通过输入明确给出并且没有引入外部的任何数据。

In terms of functionality, it’s identical to our previous example, but we’ve dramatically improved its maintainability and testability. Now we don’t have to hack around to fake `window.innerWidth` for any tests, but instead just call `mouseOnLeftSide()` with the exact arguments we need:

在功能方面，就和前面的事例展示给我们的一样，但是我们的改善显著的提高了可维护性和可测试性。现在我们不需要为那个 `window.innerWidth` 进行测试，我们只需要调用有明确参数的 `mouseOnLeftSide()`：

```javascript
mouseOnLeftSide(5, 499) // 确保它的工作原理并且宽度小于500
```

#### __SELF-DOCUMENTING__
#### __自记录__

Besides being easier to test, pure functions have other characteristics that make them worth using whenever possible. By their very nature, pure functions are self-documenting. If you know that a function doesn’t reach out of its scope to get data, you know the only data it can possibly touch is passed in as arguments. Consider the following function definition:

除了被更容易测试，纯函数有其他特性使他们值得使用尽可能。因为他们很自然，所以纯函数是自记录。如果函数没有使用自身作用域外的数据，那你可以明确的知道唯一的数据来源是传参。请考虑以下函数定义：

```javascript
function mouseOnLeftSide(mouseX, windowWidth)
```

You know that this function deals with two pieces of data, and if the arguments are well named it should be clear what they are. We all have to deal with the pain of revisiting code that’s lain untouched for six months, and being able to regain familiarity with it quickly is a key skill.

你知道这个函数有两个命名明确的数据。我们将在未来大半年的时间里会重温这些代码，能够快速的熟悉他们是个关键技能。

#### __AVOIDING GLOBALS IN FUNCTIONS__
#### __避免全局函数__

The problem of global variables is well documented in JavaScript—the language makes it trivial to store data globally where all functions can access it. This is a common source of bugs, too, because anything could have changed the value of a global variable, and hence the function could now behave differently.

在JavaScript中全局变量是很好记录的。语言的特点使得他琐碎的仓储着全局数据，这样所有的函数都可以访问他。这是一切错误的根源，因为任何地方都可以改变全局变量的值，因此函数可能表现出不同的行为。

An additional property of pure functions is referential transparency. This is a rather complex term with a simple meaning: given the same inputs, the output is always the same. Going back to `mouseOnLeftSide`, let’s look at the first definition we had:

一个额外的属性的引用对于纯函数来说是透明的。这个术语相当复杂，简而言之：给相同的参数，总是会有相同的结果。回去看看 `mouseOnLeftSide` 函数，第一次我们是怎么定义他的：

```javascript
function mouseOnLeftSide(mouseX) {
    return mouseX < window.innerWidth / 2;
}
```

This function is not referentially transparent. I could call it with the input 5 multiple times, resize the window between calls, and the result would be different every time. This is a slightly contrived example, but functions that return different values even when their inputs are the same are always harder to work with. Reasoning about them is harder because you can’t guarantee their behavior. For the same reason, testing is trickier, because you don’t have full control over the data the function needs.

这个函数是不透明的。无论我调试多少次，每当窗口大小发生变化，函数都会输出不同的结果。这还是一个简单的例子，但是输入相同值却返回不同值的函数总是难以使用的。理解他们更难，因为你不能保证他们的行为。同样的原因，测试很变得棘手，因为你没有完全控制功能需要的数据。

On the other hand, our improved mouseOnLeftSide function is referentially transparent because all its data comes from inputs and it never reaches outside itself:

另一方面，我们改进的 `mouseOnLeftSide` 函数使其透明，因为它的所有数据都来自于参数，并且它永远不会调用函数外部数据：

```javascript
function mouseOnLeftSide(mouseX, windowWidth) {
    return mouseX < windowWidth / 2;
}
```

You get referential transparency for free when following the rule of declaring all your data as inputs, and by doing this you eliminate an entire class of bugs around side effects and functions acting unexpectedly. If you have full control over the data, you can hunt down and replicate bugs much more quickly and reliably without chancing the lottery of global variables that could interfere.

遵循将所有数据都由参数声明的规则，你可以很容易获得清晰的参考，这样做可以消除由于副作用函数带来的出人意料的错误。如果你完全控制数据，你可以更快速，更可靠地追踪和复制错误，而无需更改可能会干扰的全局变量。

## __Choosing which functions to make pure__
## __哪些函数要变为纯函数__

It’s impossible to have pure functions consistently—there will always be a time when you need to reach out and fetch data, the most common example of which is reaching into the DOM to grab a specific element to interact with. It’s a fact of JavaScript that you’ll have to do this, and you shouldn’t feel bad about reaching outside of your function. Instead, carefully consider if there is a way to structure your code so that impure functions can be isolated. Prevent them from having broad effects throughout your codebase, and try to use pure functions whenever appropriate.

不可能总是使用纯函数的––毕竟需要一个函数来实现获取外部数据，其最常见的例子是DOM特定元素的获取和元素之进行交互。这是JavaScript的一个事实，你必须这样做，你的函数获取外部数据也不要感到不好。相反，请仔细考虑是否有办法构建你的代码，以使不纯的函数被隔离。防止它们在整个代码库中产生广泛的影响，并尽可能适当地使用纯函数。

Let’s take a look at the code below, which grabs an element from the DOM and changes its background color to red:

让我们看看下面的代码，从DOMA中获取一个元素并且把他的背景色改为红色：

```javascript
function changeElementToRed() {
    var foo = document.getElementById('foo');
    foo.style.backgroundColor = "red";
}
changeElementToRed();
```

There are two problems with this piece of code, both solvable by transitioning to a pure function:

这行代码中有两个问题，都可以通过转换为纯函数来解决：

This function is not reusable at all; it’s directly tied to a specific DOM element. If we wanted to reuse it to change a different element, we couldn’t.

这个函数根本不可复用；它直接绑定到一个特定的DOM元素。如果我们想复用它来改变一个不同的元素，那就不可以了。

This function is hard to test because it’s not pure. To test it, we would have to create an element with a specific ID rather than any generic element.

这也不是个纯函数所以很难测试，要测试他，我们必须创建一个有特定id的元素而不是一个通用元素。

Given the two points above, I would rewrite this function to:

就上面讲到的两点，我们需要重写函数：

```javascript
function changeElementToRed(elem) {
    elem.style.backgroundColor = "red";
}

function changeFooToRed() {
    var foo = document.getElementById('foo');
    changeElementToRed(foo);
}

changeFooToRed();
```

We’ve now changed `changeelementtored()` to not be tied to a specific DOM element and to be more generic. At the same time, we’ve made it pure, bringing us all the benefits discussed previously.

我们现在重写了 `changeelementtored()`，他不被捆绑到一个特定的DOM元素变得更通用。这样做，我们使他变得更纯粹，并且都来了我们先前所讨论的好处。

It’s important to note, though, that I’ve still got some impure code `changeFooToRed()` is impure. You can never avoid this, but it’s about spotting opportunities where turning a function pure would increase its readability, reusability, and testability. By keeping the places where you’re impure to a minimum and creating as many pure, reusable functions as you can, you’ll save yourself a huge amount of pain in the future and write better code.

注意！我们的代码中还有不纯粹的地方，比如 `changeFooToRed()`，就是不纯粹的。在项目中不纯粹的代码是不可避免的，但是我们要尽可能的把他们变得纯粹，因为这样可以提高代码的可读性，可重用性和可测试性。把不纯粹的代码保持在最低限度，并尽可能多的创建可复用的纯粹函数可，您将在以后为自己减少痛苦，并可以编写出更好的代码。

## __Conclusion__
## __结论__

“Pure functions,” “side effects,” and “referential transparency” are terms usually associated with purely functional languages, but that doesn’t mean we can’t take the principles and apply them to our JavaScript, too. By being mindful of these principles and applying them wisely when your code could benefit from them you’ll gain more reliable, self-documenting codebases that are easier to work with and that break less often. I encourage you to keep this in mind next time you’re writing new code, or even revisiting some existing code. It will take some time to get used to these ideas, but soon you’ll find yourself applying them without even thinking about it. Your fellow developers and your future self will thank you.

“纯函数”，“副作用”和“参照透明度”通常与纯函数式语言相关联，但这并不意味着我们不能将这些原则应用到JavaScript中。通过注意这些原则并合理地应用它们，当你的的代码从中受益时，你将获得更多的可靠性。我鼓励你在下一次写新的代码时参考这些原则，甚至重新审视一些现有的代码。当然习惯这些想法需要一些时间，但很快你会发现自己抛弃原来的顾虑应用它们。你的同事和未来的你会为此感谢你。