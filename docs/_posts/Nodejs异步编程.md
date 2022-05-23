---
title: js异步编程
date: 2020-07-01 09:47:14
tags:
  - Node.js
  - Promise
categories:
  - JavaScript
permalink: /pages/36e995/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

## 同步 api,异步 api

同步 API:只有当前 API 执行完成后,才能继续执行下一个 API

```js
console.log('before');
console.log('after');
//输出before after
```

异步 API:当前 API 的执行不会阻塞后续代码的执行

```js
console.log('before') ;
setTimeout (
    () => { console.log('last') ;
}，2000) ;
console. log('after') ;
//输出after before
```

## 同步 api，异步 api 的区别（获取返回值）

同步 API 可以从返回值中拿到 API 执行的结果，但是异步 API 是不可以的

```js
//同步
function sum (n1, n2) {
    return n1 + n2;
}
const result = sum (10， 20) ;
//输出30
//异步
function getMsg() {
    setTimeout (function() {
        return { msg: 'Hello Node. js' }
    }，2000) ;
const msg = getMsg() ;
//msg=undefined
```

那么，异步 api 的返回值要怎么拿到呢？可以通过回调函数

## 回调函数

自己定义，让别人去调用
来看一个简单的栗子

```js
function getData(callback) {
  callback();
}
getData(function () {
  console.log('callback函数被调用了');
});
```

把函数当做参数传递，我们还可以给传递的函数再加一个参数

```js
function getData(callback) {
  callback('123');
}
getData(function (n) {
  console.log('callback函数被调用了');
  console.log(n);
});
```

&emsp;&emsp;**如果 getData 这个函数内部有异步操作，那么在异步操作执行完成的时候，就会调用回调函数，并且把异步 api 执行的结果通过参数的形式传递给 callback，那么在 getData 里面的回调函数里面就能拿到这个异步 api 执行的结果**
来看一个定时器的异步例子

```js
function getMsg(callback) {
  setTimeout(function () {
    callback({
      msg: 'hello node.js',
    });
  }, 2000);
}
getMsg(function (data) {
  console.log(data);
});
```

## 异步编程代码执行顺序分析

```js
console.log('代码开始执行');

setTimeout(function () {
  console.log('2s');
}, 2000);

setTimeout(function () {
  console.log('0s');
}, 0);

console.log('代码结束执行');
//输出结果：代码开始执行  代码结束执行 0s 2s
```

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/代码执行顺序.png)

当同步代码执行结束后，开始执行异步代码，0 秒执行的定时器先执行完，系统调用他的回调函数

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/代码执行顺序1.png)

然后，2 秒钟之后执行的定时器也执行完，加入到执行队列中

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/代码执行顺序2.png)

大功告成！

## Node.js 中的异步 api

```js
fs.readFile('./demo. txt',(err, result) =>{}) ;

var server = http. createServer() ;
server.on(' request'，(req, res) => {}) ;
```

&emsp;&emsp;如果异步 API 后面代码的执行依赖当前异步 API 的执行结果，但实际上后续代码在执行的时候异步 API 还没有返回结果，这个问题要怎么解决呢?

现在，我们有个需求：依次读取 A 文件、B 文件、C 文件

```js
const fs = require('fs' );

fs. readFile('./1.txt'，' utf8', (err, result1) => {
    console.log(result1 )
    fs .readFile('./2.txt', 'utf8', (err, result2) => {
        console. log(result2)
        fs.readFile('./3.txt', ' utf8', (err, result3) =>
            console . log(result3)
        })

    })
});
```

如果我们无限嵌套下去，发生了回调地狱。ES6 为我们提供了一个 promise 的方法来解决回调地狱

## Promise

Promise 出现的目的是解决 Nodejs 异步编程中回调地狱的问题。

```js
let promise = new Promise((resolve, reject) => {
    setTimeout(() => {
        if (true) {
            resolve ({name: '张三' })
        }else {
            reject('失败了')
        }
    }，2000) ;
}) ;
```

- 当异步 api 有返回结果时，可以调用 resolve 函数，并把异步 api 的执行结果通过参数形式传递
- 当异步 api 执行失败，我们就可以调用 reject 这个函数，把这个失败的结果传递到 promise 外面

我们在 promise 外面怎么拿到异步 api 的执行结果呢？

```js
promise.then (result => console.log(result) ; // (name: '张三'}
    .catch(error => console.log(error) ;// 失败了)
```

对 promise 有个了解后，我们来实现需求,先读取一个文件

```js
const fs = require('fs');
let promise = new Promise((resolve, reject) => {
  fs.readFile('./1. txt', 'utf8', (err, result) => {
    if (err != null) {
      reject(err);
    } else {
      resolve(result);
      //异步api执行成功后调用resolve，其实就是调用then里面的()=>{}
    }
  });
});
promise
  .then((result) => {
    console.log(result);
  })
  .catch((err) => {
    console.log(err);
  });
```

我们改造一下，利用 promise 解决回调地狱

```js
const fs = require('fs' );

function p1(){
    return new Promise((resolve,reject) =>{
        fs.readFile('./1.txt'，' utf8', (err, result1) => {
            resolve(result1)
             //异步api执行成功后调用resolve，其实就是调用then里面的()=>{}
        })
    })
}
function p2(){
    return new Promise((resolve,reject) =>{
        fs.readFile('./2.txt'，' utf8', (err, result1) => {
            resolve(result2)
        })
    })
}
function p3(){
    return new Promise((resolve,reject) =>{
        fs.readFile('./3.txt'，' utf8', (err, result1) => {
            resolve(result3)
        })
    })
}

p1().then((result1)=>{
    console.log(result);
    return p2();//这里的p2是一个promise对象，相当于return了一个promise对象，然后对这个promise对象调用.then方法
})
.then((result2)=>{
    console.log(result2)
    return p3();
})
.then((result3)=>{
    console.log(result3)
})
```

## 异步函数

异步函数是 ES7 中新增方法，异步函数是异步编程语法的终极解决方案,它可以让我们将异步代码写成同步的形式，让代码不再有回调函数嵌套,使代码变得清晰明了。
**async 关键字**

- 在普通函数前面加上 async 关键字，普通函数就变成了异步函数
- 异步函数默认返回值是 promise 对象
- 在异步函数内部使用 return 关键字进行结果返回结果会被包裹的 promise 对象中 return 关键字代替了 resolve 方法
- 在异步函数内部使用 throw 关键字抛出程序异常
- 调用异步函数再链式调用 then 方法获取异步函数执行结果
- 调用异步函数再链式调用 catch 方法获取异步函数执行的错误信息

**await 关键字**

- 它只能出现在异步函数中
- await promise 它可以暂停异步函数的执行等待 promise 对象返回结果后再向下执行
- await promise await 后面只能写 promise 对象，写其他类型的 API 是不可以的

```js
async function p1 (){
    return 'p1';
    //async修饰的函数返回一个promise对象
    //return promise{'p1'}
}
async function p2 (){
    return 'p2';
}
async function p3 (){
    return 'p3';
}
async function run() {
    let r1 = await p1()
    let r2 = await p2()
    let r3 = await p3()
    console.log(r1)
    console.1og(r2)
    console.1og(r3)
}
run()
//输出 p1 p2 p3 把异步操作写成了同步代码
```

现在，我们用 async、await 改造读取文件顺序输出需求

&emsp;&emsp;首先，我们知道，`fs.readFile()`方法是通过返回值的方法来获取文件的读取结果，也就是说，他不返回 promise 对象，没办法加 async/await 关键字,后来，nodejs 为我们提供了一个`promisify`方法，可以对现有的异步 api 进行包装，让方法返回 promise 对象，以支持异步函数语法

> const promisify = require('util').promisify

我们开始

```js
const fs = require('fs');
//改造现有异步函数api让其返回promise对象从而支持异步函数语法
const promisify = require('util').promisify;
const readFile = promisify(fs.readFile);
async function run() {
  let r1 = await readFile(' ./1.txt', 'utf8');
  let r1 = await readFile(' ./1.txt', 'utf8');
  let r1 = await readFile(' ./1.txt', 'utf8');
  console.log(r1);
  console.log(r2);
  console.log(r3);
}
run();
```
