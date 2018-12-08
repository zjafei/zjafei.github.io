---
title: react通过cloneElement修改props.children的props属性
date: 2018-12-08 11:44:49
tags: ['react','cloneElement','children','props']
category: 'coding'
---

### 应用场景

在`Parent`内部把值传递给`Children`。
```javascript
<Parent>
  <Children/>
</Parent>
```
一般是通