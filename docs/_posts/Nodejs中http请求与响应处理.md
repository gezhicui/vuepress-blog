---
title: Nodejs中http请求与响应处理
date: 2020-06-30 15:47:38
tags:
  - Node.js
categories:
  - NodeJs
permalink: /pages/a36e52/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

首先，我们先来创建一个网站服务器

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/node创建网站服务器.png)

其中，当有请求来的时候，会触发 request 这个事件，然后执行后面的事件处理函数。我们可以通过 res.end 对客户端进行响应

## 请求报文

### 请求方式

- Get 请求数据
- Post 请求数据

### 请求地址

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/请求地址.png)
我们可以通过`req.url`来获取客户端请求地址，通过`req.headers`获取请求报文，`req.methods`获取请求方法(get/post)

## 响应报文

### http 状态码

- 200 请求成功
- 404 请求的资源没有被找到
- 500 服务器端错误
- 400 客户端请求有语法错误

### 响应内容类型

- text/html
- text/css
- application/javascript
- image/jpeg
- application/json

## 请求参数

客户端向服务器端发送请求时，有时需要携带一些客户信息, 客户信息需要通过请求参数的形式传递到服务器端，比如登录操作。

### get 请求参数

- 参数被放置在浏览器地址栏中，例如: http://localhost:3000/?name= zhangsan&age= 20

那么怎么拿到 get 的参数呢？
node 为我们提供了一个内置模块

> const url = require('url)

在 url 模块中，有一个 parse()方法可以处理 url，我们只需要将想解析的 url 传给 url.parse()方法进行解析，最终，这个方法会返回一个对象。我们来 console.log 一下`url.parse(req.url)`

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/urlparse方法.png)

现在，我们把查询参数转换成对象。

> url.parse(req.url,true)

- 第一个参数是要解析的 url 地址
- 第二个参数表示把查询参数解析成对象形式。

此时，url.paesr 中的 query 参数会变成一个对象 `query:'name=zhangsan ,age='20'`
这个时候，我们就可以通过`url.parse(res.url,true).query`拿到这个对象

> let params = url.parse(res.url,true).query

```js
cocnsole.log(params.name); //zhangsan
cocnsole.log(params.age); //20
```

在 parse()方法中，还有一个 pathname 属性可以帮我们获取到不包含请求参数的请求地址。这样，我们就可以通过对象结解构的方式拿到 query 和 pathname

> let {query, pathname} = url.parse(res.url,true)

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/parse方法.png)

### post 请求参数

我们可以通过表单方式提交一个 post 请求，post 方式是通过事件的方式接收的。**当请求传递参数时触发 data 事件**，**当传递完成时触发 end 事件**

我们通过 querystring 这个内置模块来处理 post 的请求参数

> const querystring = require('querystring');

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/querystring方法处理post.png)

## 路由

http://localhost:3000/index

http://localhost:3000/login

路由是指客户端请求地址与服务器端程序代码的对应关系。简单的说，就是请求什么响应什么。

实现路由的核心代码

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/实现路由的核心代码.png)
