---
title: Promise的并发控制
date: 2022-05-22 22:35:46
tags:
  - Promise
  - 异步编程
categories:
  - JavaScript
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
permalink: /pages/4951e7/
---

## 背景

当我们的应用在瞬间发出很多请求,例如几十万 http 请求（tcp 连接数不足可能造成等待），或者堆积了无数调用栈导致内存溢出,这个时候需要我们对 http 的连接数做限制。在有限的并发数下尽快完成所有请求

<!-- more -->

## 思路

思路如下：初始化一个 `pool`数组 作为并发池，然后先循环把并发池塞满，不断地调用 `addTask` 然后通过自己自定义的请求函数`requst`(请求函数可以是网络请求封装的 promise 对象，或者是其他的)，每个任务`task`是一个**Promise 对象包装的**，执行完就 `pop` 出连接池， 然后将新任务`push` 添加进并发池 `pool` 中。

## 实现

### 方式 1-不通过 Promise.race

```js
//自定义请求函数
var request = (url) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(`任务${url}完成`);
    }, 1000);
  }).then((res) => {
    console.log('外部逻辑', res);
  });
};

//添加任务
function addTask(url) {
  let task = request(url);
  pool.push(task);
  task.then((res) => {
    //请求结束后将该Promise任务从并发池中移除
    pool.splice(pool.indexOf(task), 1);
    console.log(`${url} 结束，当前并发数：${pool.length}`);
    url = urls.shift();
    // //每当并发池跑完一个任务，就再塞入一个任务
    if (url !== undefined) {
      addTask(url);
    }
  });
}

let urls = [
  'bytedance.com',
  'tencent.com',
  'alibaba.com',
  'microsoft.com',
  'apple.com',
  'hulu.com',
  'amazon.com',
]; // 请求地址
let pool = []; //并发池
let max = 3; //最大并发量
//先循环把并发池塞满
while (pool.length < max) {
  let url = urls.shift();
  addTask(url);
}
```

### 方式 2-通过 Promise.race

```js
//自定义请求函数
var request = (url) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(`任务${url}完成`);
    }, 1000);
  }).then((res) => {
    console.log('外部逻辑', res);
  });
};

//添加任务
function addTask(url) {
  let task = request(url);
  pool.push(task);
  task.then((res) => {
    //请求结束后将该Promise任务从并发池中移除
    pool.splice(pool.indexOf(task), 1);
    console.log(`${url} 结束，当前并发数：${pool.length}`);
  });
}
//每当并发池跑完一个任务，就再塞入一个任务
function run(race) {
  race.then((res) => {
    let url = urls.shift();
    if (url !== undefined) {
      addTask(url);
      run(Promise.race(pool));
    }
  });
}

let urls = [
  'bytedance.com',
  'tencent.com',
  'alibaba.com',
  'microsoft.com',
  'apple.com',
  'hulu.com',
  'amazon.com',
]; // 请求地址
let pool = []; //并发池
let max = 3; //最大并发量
//先循环把并发池塞满
while (pool.length < max) {
  let url = urls.shift();
  addTask(url);
}
//利用Promise.race方法来获得并发池中某任务完成的信号
let race = Promise.race(pool);
run(race);
```

### 方式 3-通过 Promise.race 和异步函数

```js
//自定义请求函数
var request = (url) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(`任务${url}完成`);
    }, 1000);
  }).then((res) => {
    console.log('外部逻辑', res);
  });
};
// 执行任务
async function fn() {
  let urls = [
    'bytedance.com',
    'tencent.com',
    'alibaba.com',
    'microsoft.com',
    'apple.com',
    'hulu.com',
    'amazon.com',
  ]; // 请求地址
  let pool = []; //并发池
  let max = 3; //最大并发量
  for (let i = 0; i < urls.length; i++) {
    let url = urls[i];
    let task = request(url);
    task.then((data) => {
      //每当并发池跑完一个任务,从并发池删除个任务
      pool.splice(pool.indexOf(task), 1);
      console.log(`${url} 结束，当前并发数：${pool.length}`);
    });
    pool.push(task);
    if (pool.length === max) {
      //利用Promise.race方法来获得并发池中某任务完成的信号
      //跟await结合当有任务完成才让程序继续执行,让循环把并发池塞满
      await Promise.race(pool);
    }
  }
}
fn();
```
