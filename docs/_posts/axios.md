---
title: axios
date: 2020-08-11 16:53:41
tags: 前端
categories: 前端
permalink: /pages/b460c6/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

axios(官网: https://github.com/axios/axios )是一个基于 Promise 用于浏览器和 node.js 的 HTTP 客户端。
它具有以下特征:

- 支持浏览器和 node.js 心
- 支持 promise
- 能拦截请求和响应
- 自动转换 JSON 数据

## 安装

> npm install axios -S

## 在 main.js 中导入

```js
import Vue from 'vue';
import axios from 'axios';
Vue.prototype.$axios = axios; //全局注册，使用方法为:this.$axios
Vue.prototype.qs = qs; //全局注册，使用方法为:this.qs
```

## axios 的基本用法

```js
axios.get('/data').then((res) => {
  // data属性名称是固定的，用于获取后台响应的数据
  console.log(res.data);
});
```

## axios 常用 API

- get :查询数据
- post:添加数据
- put :修改数据
- delete:删除数据

## axios 的参数传递

### get 参数传递

- 通过 URL 传递参数
- 通过 params 选项传递参数

```js
//通过url传参(第一种)
axios.get( '/adata?id=123')
    .then (res=>{
        console.log (res.data)
    })

//接口实现
app.get('/adata', (req,res)=>{
    res.send("axios get 传递参数"+ req.query.id)
})

//通过url传参(第二种)
axios.get( '/adata/123')
    .then (res=>{
        console.log (res.data)
    })
//接口实现
app.get('/adata/:id', (req，res) =>{
res.send('axios get (Restful）传递参数'+req.params.id)
}

//通过params属性传参（params用于传递get参数）
axios.get('/adata',{
    params : {
        id: 123
    }
})
.then(res=>{
    console.log (res.data)
})
```

### delete 参数传递

传递方式与 get 类似 也是通过 url 和 param，我这里实现一下 params

```js
//通过params属性传参
axios.delete('/adata',{
    params : {
        id: 123
    }
})
.then(res=>{
    console.log (res.data)
})
//接口实现
app.delete('/axios',(req，res)=>{
    res.send('axios get传递参数'+req.query.id)
})
```

### post 参数传递

- 通过选项传递参数（默认传递的是 json 格式的数据）
- 通过 URLSearchParams 传递参数(application/x-www-form-urlencoded)(了解就好)

```js
axios
  .post('/adata', {
    uname: 'tom',
    pwd: 123,
  })
  .then((res) => {
    console.log(res.data);
  });
//接口实现
app.post('/axios', (req, res) => {
  res.send('axios post传递参数' + req.body.uname + '---' + req.body.pwd);
});
```

### put 参数传递

与 post 类似 但是一般会在 url 后面跟上具体要修改的内容的 id

```js
axios
  .post('/adata/123', {
    //123为要修改的信息的id
    uname: 'tom', //要修改的对应id的信息
    pwd: 123,
  })
  .then((res) => {
    console.log(res.data);
  });
//接口实现
app.post('/axios/:id', (req, res) => {
  res.send(
    'axios put传递参数' +
      req.params.id +
      '---' +
      req.body.uname +
      '---' +
      req.body.pwd
  );
});
```

## axios 的全局配置

- axios.defaults.timeout = 3000//超时时间
- axios.defaults.baseURL = 'http://localhost:3000/app';//默认地址
- axios.defaults.headers['mytoken']= 'aqwerwqwerqwer2ewrwe23eresdf23’//设置请求头

## axios 拦截器

### 请求拦截器

在请求发出之前设置一些信息
![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/请求拦截器.png)

```js
//添加一个请求拦截器
axios.interceptors.request.use(
  function (config) {
    //在请求发出之前进行一些信息设置
    console.log(config.url); //判断url需不需要加请求头
    config.headers.mytoken = 'nihao';
    return config;
  },
  function (err) {
    //处理响应的错误信息
  }
);
```

### 响应拦截器

在获取数据之前对数据做一些加工处理
![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/响应拦截器.png)

```js
//添加一个响应拦截器
axios.interceptors.response.use(
  function (res) {
    //在这里对返回的数据进行处理 这里把data拆解出来
    var data = res.data;
    return data;
  },
  function (err) {
    //处理响应的错误信息
  }
);
```

## 异步终极解决方案 async/await

- async/await 是 ES7 引入的新语法，可以更加方便的进行异步操作
- async 关键字用于函数上(async 函数的返回值是 Promise 实例对象)
- await 关键字用于 async 函数当中(await 可以得到异步的结果)

```js
async function queryData() {
  var res = await axios.get('adata'); //不需要调用then和回调函数，await可以直接返回结果
  console.log(res.data);
}
queryData();

//如果你想把结果中的res.data return出去，那么这个return出去的东西是一个peomise对象，可供后面链式调用

async function queryData() {
  var res = await axios.get('adata'); //不需要调用then，await直接返回结果
  return res.data;
}
queryData().then((data) => {
  consolr.log(data);
});
```

await 后面需要跟上一个异步对象，如果是我们自己写的 promise 对象想用 async 和 await，如下

```js
async function queryData() {
  var res = await axios.get('adata'); //不需要调用then，await直接返回结果
  return res.data;
}
queryData().then((data) => {
  consolr.log(data);
});
```
