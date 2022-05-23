---
title: async await原理及简单实现
date: 2022-05-17 16:39:00
tags:
  - JavaScript
  - 异步编程
categories:
  - JavaScript
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
permalink: /pages/64d2f0/
---

解决函数回调经历了几个阶段， Promise 对象， Generator 函数到 async 函数。async 函数目前是解决函数回调的最佳方案。很多语言目前都实现了 async，包括 Python ，java spring，go 等。

<!-- more -->

## async await 的用法

async 函数**返回一个 Promise 对象**，当函数执行的时候，一旦遇到 await 就会先返回，等到触发的异步操作完成，再接着执行函数体内后面的语句。

```js
function getNum(num) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(num + 1);
    }, 1000);
  });
}
const func = async () => {
  const f1 = await getNum(1);
  const f2 = await getNum(f1);
  console.log(f2);
  // 输出3
};
func();
```

async /await 需要在 function 外部书写 async，在内部需要等待执行的函数前书写 await 即可

## 实现一个简单的 async/await

理解 async 函数需要先理解 Generator 函数，因为 async 函数是 Generator 函数的语法糖。关于 Generator 函数在[迭代器和生成器](/pages/5ec76d/)一文中有写到

async/await 语法糖就是使用 Generator 函数+自动执行器来运作的。 我们可以参考以下例子

```js
// 定义了一个promise，用来模拟异步请求，作用是传入参数++
function getNum(num) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(num + 1);
    }, 1000);
  });
}

//自动执行器，如果一个Generator函数没有执行完，则递归调用
function asyncFun(func) {
  var gen = func();
  function next(data) {
    var result = gen.next(data);
    if (result.done) return result.value;
    result.value.then((data) => next(data));
  }
  next();
}

// 所需要执行的Generator函数，内部的数据在执行完成一步的promise之后，再调用下一步
var func = function* () {
  var f1 = yield getNum(1);
  var f2 = yield getNum(f1);
  console.log(f2);
};
asyncFun(func);
```

在执行的过程中，判断一个函数的 promise 是否完成，如果已经完成，将结果传入下一个函数，继续重复此步骤。

## 注意点

**async/await 不能阻塞返回非 Promise 对象**

```js
async function fn1() {
  console.log(1);
  await pr1();
  await pr2();
  console.log(2);
}

function fn2() {
  console.log(3);
}

function pr1() {
  //由于pr1不是返回promise，那么它就按正常的js顺序执行，先执行同步代码，当主线程空闲了，再去执行异步队列的任务
  setTimeout(() => {
    console.log(4);
  }, 2000);
}

function pr2() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log(5);
      resolve();
    }, 1000);
  });
}

(async () => {
  await fn1();
  fn2();
})();

//输出15234
```

如果我们把上面的 pr1 改成返回 promise，结果就不一样了

```js
async function fn1() {
  console.log(1);
  await pr1();
  await pr2();
  console.log(2);
}

function fn2() {
  console.log(3);
}

function pr1() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log(4);
      resolve();
    }, 2000);
  });
}

function pr2() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log(5);
      resolve();
    }, 1000);
  });
}

(async () => {
  await fn1();
  fn2();
})();

//14532
```
