---
title: 使用URLSearchParams对URL?传参进行处理
date: 2021-04-30 17:36:23
tags:
  - JavaScript
categories:
  - JavaScript
permalink: /pages/eb3f83/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

在最近遇到的项目中，在传递一些有时候需要有时候不需要的参数时，我们就能用`?`来进行 URL 的传参，这样并不改变路由的跳转路径，比如说：

> http://localhost:8000/#/aaa

> http://localhost:8000/#/aaa?id=3

> http://localhost:8000/#/aaa?id=3&name=xixi

这三个路由都会匹配到/aaa 的路由中去。这也方便了一些场景下的开发，但是头疼的是获取参数就很不方便，第三个 URL 我们通过 history 进行获取到的结果如下：

```js
const { search } = this.props.location;
console.log(search); //?id=3&name=xixi
```

这样我们获取传递的参数就很麻烦，好在现在的标准支持了`URLSearchParams`，我们使用这个接口方法来实现一下对 URL 参数的获取:

```js
const { search } = this.props.location;
const paramsString = search.substring(1);
const searchParams = new URLSearchParams(paramsString);
const id = searchParams.get('id');
const name = searchParams.get('name');
```

这样，我们就获取到了 id 和 name 的值
