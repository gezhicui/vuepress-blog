---
title: Vue路由传参param与query两种方式的区别
date: 2020-07-28 22:08:58
tags:
  - Vue
categories:
  - 前端
  - Vue
copyright: true
permalink: /pages/34053a/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

## 基本说明

`query`传参是针对`path`的，`params`传参是针对`name`的。接收参数的方式都差不多，通过`this.$route.query`和`this.$route.params`

路由配置：

```js
{path:'/login',name:'Login',component:Login},
```

1.页面携带 query 参数跳转(path,name 指定跳转到 Login 时都可以携带 query 参数)

```js
this.$router.push({ path:'/login',name:'Login', query: { id: this.id } )
```

query 相当与发送了一次 get 请求，请求参数会显示在浏览器地址栏中

2.页面携带 params 参数跳转(携带 params 参数跳转时只能使用 name 指定)

```js
this.$router.push({ name:'Login', params: { id: this.id } )
```

params 相当与发送了一次 post 请求，请求参数则不会显示，并且刷新页面之后参数会消失

当路由配置更改为

路由配置：

```js
{path:'/login/:id',name:'Login',component:Login}
```

并且再次发送请求，请求数据不会随着页面的刷新而消失

获取请求参数

```js
this.$route.params.id;
this.$route.query.id;
```

注:

&emsp;&emsp;`router`是`VueRouter` 的一个对象，通过`Vue.use(VueRouter)`和`VueRouter`构造函数得到一个`router`的实例对象，这个对象中是一个全局的对象，他包含了所有的路由包含了许多关键的对象和属性。

&emsp;&emsp;`$router.push({path:'login'})`;本质是向`history`栈中添加一个路由，在我们看来是 切换路由，但本质是在添加一个`history`记录;

&emsp;&emsp;而`route`是一个跳转的路由对象，每一个路由都会有一个`route`对象，是一个局部的对象，可以获取对应的`name,path,params,query`等

## 两者差别

### 用法上的

&emsp;&emsp;`query`要用`path`来引入，`params`要用`name`来引入，接收参数都是类似的，分别是`this.$route.query.name`和`this.$route.params.name`。

注意接收参数的时候，已经是`$route`而不是`$router`了哦！！

### 展示上的

&emsp;&emsp;query 更加类似于我们 ajax 中 get 传参，params 则类似于 post，说的再简单一点，前者在浏览器地址栏中显示参数，后者则不显示

query:
![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/query路由.png)
params:  
![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/params路由.png)
