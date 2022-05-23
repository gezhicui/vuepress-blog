---
title: Cookie和Session的使用
date: 2020-07-22 18:21:28
tags:
  - 前端
categories:
  - 前端
permalink: /pages/025a60/
sidebar: auto
author:
  name: 杨雨翔
  link: https://gitee.com/xiang0515
---

## cookie 和 session 概念

### Cookie

&emsp;&emsp;cookie 是浏览器在电脑硬盘中开辟的一块空间，主要供服务器端存储数据。

- cookie 中的数据是以域名的形式进行区分的。
- cookie 中的数据是**有过期时间**的，超过时间数据会被浏览器**自动删除**。
- cookie 中的数据会随着请求被**自动发送**到服务器端。
- 单个 cookie 大小不能超过 4k，很多浏览器限制一个站点最多 20 个 cookie

### 如何查看 cookie

&emsp;&emsp;我们访问百度，在浏览器的开发者模式下 Application 中就可以查看百度给我们的 cookie
![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/百度cookie.png)

### Session

&emsp;&emsp;session 是另一种记录客户状态的机制，不同的是 Cookie 保存在客户端浏览器中，而 session 保存在服务器上。在 session 对象中也可以存储多条数据，每一条数据都有一个`sessionid`做为唯一标识。

### session 工作流程

当浏览器访问服务器并发送第一次请求时，服务器端会创建一个 session 对象，生成一
个类似于 key,value 的键值对，然后将 key(cookie)返回到浏览器(客户)端，浏览器下次
再访问时，携带 key(cookie)，找到对应的 session(value)。

### cookie 和 session 关系

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/cookie和session关系.png)

## Cookie 在 Nodejs 中使用

首先我们要安装一个模块

> npm install cookie-parser --save

然后引入并使用

```js
//引入依赖项
var express = require('express');
var cookieParser = require('cookie-parser');
//配置中间件
var app = express();
app.use(cookieParser());

//监听路由
app.get('/', (req, res) => {
  //设置cookie(名称，值，配置),保存到客户端浏览器
  res.cookie('username', 'zhangsan', {
    //设置cookie过期时间 单位：毫秒
    maxAge: 1000 * 60 * 60,
  });
  res.send('你好');
});

app.get('/', (req, res) => {
  //从客户端获取cookie
  let name = req.cookies.username;
  console.log(name);
});
```

在上面的代码中，我们发现 cookie 是可以配置参数的，在设置 cookie 的源码中定义了这几个配置项：

```js
maxAge?: number;//过期时间（毫秒数）
signed?: boolean;//是否加密  true为加密
    {   //用法 1、在配置的时候传入一个随意的加密字符串
        app.use(cookieParser('yang'))
        //2、在设置cookie时，传入一个signed:true配置项
        res.cookie('username','zhangsan'{signed:true})
        //3、在获取加密cookie时，用以下方法获取  **只有用以下方法才能获取到cookie**
        let name = req.signedCookie.username

    }
expires?: Date;//过期时间（设置具体日期）
httpOnly?: boolean;//是否只能在后台访问
path?: string;//允许访问cookie的后端路由
domain?: string;//多个域共享cookie
    //用法
    {domain:".yang.com"}//这样配置后，aaa.yang.com和bbb.yang.com都可以访问到设置的cookie
secure?: boolean;//设置为true，说明cookie在http中无效，在https中才有效
encode?: (val: string) => string;
samesite? : boolean | 'lax'| 'strict'| "none";
```

## Session 在 Nodejs 中使用

&emsp;&emsp;在 node.js 中需要借助 express-session 实现 session 功能。

> npm install express-session

```js
//导包
const express=require('express')
const session = require('express-session')
const app = express()
//配置session的中间件函数
app.use(session({
    secret :'secret key'//服务器生成session的签名（随意写）
    //以下可忽略,按照默认配置
    resave:false////强制保存session即使它并没有变化
    saveUninitialized:true，//强制将未初始化的session存储
    name:'node'//修改session对应cookie的名称
    rolling:true//在每次请求时强行设置cookie，这将重置cookie过期时间（默认: false)
    cookie{
        maxAge:1000*60
        secure:false
        //详见cookie配置
    }
    }));
```

在用户访问后端路由后，设置一个 session

```js
app.get('/login', (req, res) => {
  //设置session
  req.session.username = 'zhangsan';
  res.send('登录');
});
//获取session
```

在用户再进行登录时，判断用户登录状态

```js
//拦截请求判断用户登录状态
app.use('/admin', (req, res, next) => {
  if (req.url != '/login' && !req.session.username) {
    //判断用户访问的是否是登录页面
    //判断用户的登录状态
    //如果用户是登录的将请求放行
    //如果用户不是登录的将请求重定向到登录页面
    res.redirect(' /admin/ login');
    console.log(req.session.username);
  } else {
    //用户是登录状态将请求放行
    next();
  }
});
```

在用户退出登录时，我们需要销毁 session

```js
app.get('/loginout', (req, res) => {
  //1、设置session的过期时间为6(它会把所有的session都销毁)
  //req .session.cookie.maxAge=0
  //2、销毁session
  //req.session.destroy()
  //3、销毁指定session  *推荐
  req.session.username = '';

  res.send('退出登录');
});
```
