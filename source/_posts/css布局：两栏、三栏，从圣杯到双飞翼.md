---
title: css布局：两栏、三栏，从圣杯到双飞翼
date: 2017-11-23 22:45:40
tags: ['css布局','css','两栏','三栏','圣杯','双飞翼','bfc','块格式化上下文']
category: 'coding'
---

## 常规的垂直布局

```html
<!DOCTYPE html>
<html>

<head>
    <style>
        .header{
            background-color: red;
            height: 100px;
        }
        .bodyer{
            background-color: green;
            height: 500px;
        }
        .footer{
            background-color: yellow;
            height: 50px;
        }
    </style>
</head>

<body>
    <div class="header">
        Hello CSS layout
    </div> 
    <div class="bodyer">
        some thing
    </div> 
    <div class="footer">copyright</div>
</body>

</html>
```
<!--more-->
垂直布局就是这样的简单，因为垂直布局可以充分的利用[区块元素](https://zh.wikipedia.org/wiki/HTML元素#.E5.8D.80.E5.A1.8A.E5.85.83.E7.B4.A0)的特点就可以实现。

## 常规的水平布局
### 通过inline-block实现（inline与inline-block的区别）

```html
<!DOCTYPE html>
<html>

<head>
    <style>
        .bodyer {
            background-color: green;
            height: 500px;
        }
        .col {
            background-color: red;
            display: inline-block;
            width: 200px;
            height: 200px;
        }
    </style>
</head>

<body>
    <div class="bodyer">
        <div class="col">0</div>
        <div class="col">1</div>
        <div class="col">2</div>
        <div class="col">3</div>
        <div class="col">4</div>
        <div class="col">5</div>
        <div class="col">6</div>
        <div class="col">7</div>
        <div class="col">8</div>
        <div class="col">9</div>
    </div>
</body>

</html>
```

### 通过float

```html
<!DOCTYPE html>
<html>

<head>
    <style>
        
        .bodyer {
            background-color: green;
        }
        .col {
            background-color: red;
            float: left;
            width: 200px;
            height: 200px;
        }
    </style>
</head>

<body>
    <div class="bodyer">
        <span class="col">0</span>
        <span class="col">1</span>
        <span class="col">2</span>
        <span class="col">3</span>
        <span class="col">4</span>
        <span class="col">5</span>
        <span class="col">6</span>
        <span class="col">7</span>
        <span class="col">8</span>
        <span class="col">9</span>
    </div>
</body>

</html>
```

通过 `direction: rtl` 我们可以看出来 `display: inline-block` 只是利用了从左到右的 `普通流（normal flow）`。

### 两栏布局
#### bodyer:bfc sidebar:float content:float 

```html
<!DOCTYPE html>
<html>

<head>
    <style>
        .bodyer {
            background-color: green;
            padding-bottom: 20px;
            overflow: hidden;
        }
        .sidebar {
            background-color: red;
            float: left;
            width: 200px;
            height: 500px;
            margin-right: 10px;
        }
        .content {
            background-color: yellow;
            float: left;
            width: 720px;
            height: 500px;
        }
    </style>
</head>

<body>
    <div class="bodyer">
        <div class="sidebar">sidebar</div>
        <div class="content">content</div>
    </div>
</body>

</html>
```

+ 优点：好理解，左右的间距不取决于sidebar的宽度。
+ 缺点：sidebar content 要定宽，无法自适应。父级元素要 `bfc`。

#### sidebar:float 

```html
<!DOCTYPE html>
<html>

<head>
    <style>
        .bodyer {
            background-color: green;
            padding-bottom: 20px;
        }
        .sidebar {
            background-color: red;
            float: left;
            width: 200px;
            height: 500px;
        }
        .content {
            background-color: yellow;
            height: 500px;
            margin-left: 210px;
        }
    </style>
</head>

<body>
    <div class="bodyer">
        <div class="sidebar">sidebar</div>
        <div class="content">content</div>
    </div>
</body>

</html>
```

+ 优点：可以自适应，父级元素无须 `bfc`。
+ 缺点：左右的间距要根据 sidebar 的宽度来决定。

#### sidebar:float content:bfc

```html
<!DOCTYPE html>
<html>

<head>
    <style>
        .bodyer {
            background-color: green;
            padding-bottom: 20px;
        }
        .sidebar {
            background-color: red;
            float: left;
            width: 200px;
            height: 500px;
            margin-right: 10px;
        }
        .content {
            background-color: yellow;
            height: 500px;
            overflow: auto;
        }
    </style>
</head>

<body>
    <div class="bodyer">
        <div class="sidebar">sidebar</div>
        <div class="content">content</div>
    </div>
</body>

</html>
```

+ 优点：可以自适应，父级元素无须 `bfc`，左右的间距不取决于sidebar的宽度。
+ 缺点：bfc带的副作用

### 三栏布局
#### sidebar:float content:bfc
  
```html
<!DOCTYPE html>
<html>

<head>
    <style>
        .fl{
            float: left;
            margin-right: 10px;
        }
        .fr{
            float: right;
            margin-left: 10px;
        }
        .bodyer {
            background-color: green;
            padding-bottom: 20px;
        }
        .sidebar {
            background-color: red;
            width: 200px;
            height: 500px;
        }
        .content {
            background-color: yellow;
            height: 500px;
            overflow: auto;
        }
    </style>
</head>

<body>
    <div class="bodyer">
        <div class="sidebar fl">sidebar</div>
        <div class="sidebar fr">sidebar</div>
        <div class="content">content</div>
    </div>
</body>

</html>
```

+ 优点：可以自适应，父级元素无须 `bfc`，左右的间距不取决于sidebar的宽度。元素、css较少。
+ 缺点：sidebar会下榻 子元素的顺序有要求

#### 圣杯布局

```html
<!DOCTYPE html>
<html>

<head>
    <style>
        .fl {
            float: left;
            height: 500px;
        }

        .bodyer {
            background-color: green;
            padding-bottom: 20px;
            overflow: auto;
            padding: 0 210px;
        }

        .sidebar {
            background-color: red;
            position: relative;
            width: 200px;
        }

        .left-sidebar {
            margin-left: -100%;
            left: -210px;
        }
        .right-sidebar {
            margin-left: -200px;
            right: -210px;
        }

        .content {
            width: 100%;
            background-color: yellow;
        }
    </style>
</head>

<body>
    <div class="bodyer">
        <div class="content fl">content</div>
        <div class="sidebar fl left-sidebar">sidebar</div>
        <div class="sidebar fl right-sidebar">sidebar</div>
    </div>
</body>

</html>
```

+ 优点：可以自适应， 子元素的顺序无要求。
+ 缺点：sidebar会下榻，左右的间距取决于sidebar的宽度


#### 双飞翼布局

```html
<!DOCTYPE html>
<html>

<head>
    <style>
        .fl {
            float: left;
        }

        .bodyer {
            width: 100%;
            background-color: green;
            padding-bottom: 20px;
        }

        .content {
            width: 100%;
        }

        .content-main {
            margin-left: 210px;
            margin-right: 210px;
            background-color: yellow;
            height: 500px;
        }

        .sidebar {
            width: 200px;
            background-color: red;
            height: 500px;
        }

        .left-sidebar {
            margin-left: -100%;
        }

        .right-sidebar {
            margin-left: -200px;
        }
    </style>
</head>

<body>
    <div class="bodyer fl">
        <div class="content fl">
            <div class="content-main">
                content
            </div>
        </div>
        <div class="sidebar left-sidebar fl">sidebar</div>
        <div class="sidebar right-sidebar fl">sidebar</div>
    </div>
</body>

</html>
```

+ 优点：可以自适应， 子元素的顺序无要求，塌陷的机会更小。
+ 缺点：sidebar会下榻，左右的间距取决于sidebar的宽度


## 参考
+ [学习CSS布局](http://zh.learnlayout.com/)
+ [BFC块格式化上下文](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context)
+ [浮动元素的规则](http://blog.csdn.net/gemingzhu/article/details/51623761) 

