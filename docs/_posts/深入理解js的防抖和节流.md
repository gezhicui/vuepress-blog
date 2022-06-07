---
title: 深入理解js的防抖和节流
date: 2020-06-14 10:32:45
tags:
  - JavaScript
categories:
  - JavaScript
permalink: /pages/cf004c/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

&emsp; 防抖和节流严格算起来应该属于性能优化的知识，但实际上遇到的频率相当高，在进行窗口的 resize、scroll，输入框内容校验等操作时，如果事件处理函数调用的频率无限制，处理不当或者放任不管就容易会加重浏览器和服务器的负担，导致用户体验非常糟糕。此时我们可以采用 debounce（防抖）和 throttle（节流）的方式来减少调用频率，同时又不影响实际效果。

## 函数防抖

`函数防抖（debounce）`：当**持续触发事件**时，一定时间段内没有再触发事件，事件处理函数才会执行一次，如果设定的时间到来之前，又一次触发了事件，就**重新开始延时**。

一起来实现个简单的 debounce

### 防抖 debounce 代码：

```javascript
window.onload = function () {
  //获取按钮并绑定事件
  var myDebounce = document.getElementById('debounce');
  myDebounce.addEventListener('click', debounce(sayDebounce));
};
//防抖功能函数，接受传参
function debounce(fn) {
  //创建一个标记用来存放定时器的返回值
  let timeout = null;
  return function () {
    //每次当用户点击、输入的时候，把前一个定时器消除
    clearTimeout(timeout);
    //创建一个新的setTimeout，这样能保证点击按钮后的间隔内，
    //如果用户还点击的话，就不会执行fn函数
    timeout = setTimeout(() => {
      fn.call(this, arguments);
    }, 1000);
  };
}
//防抖事件的处理
function sayDebounce() {
  // ...有些需要防抖的工作，在这里进行
  console.log('防抖成功~');
}
```

## 函数节流

`函数节流（throttle）`：当**持续触发事件**时，保证一定时间段内只调用一次事件处理函数。节流通俗解释就比如我们水龙头放水，阀门一打开，水哗哗的往下流，秉着勤俭节约的优良传统美德，我们要把水龙头关小点，最好是如我们心意按照一定规律在某个时间间隔内一滴一滴的往下滴。如**持续触发 scroll 事件时，并不立即执行 scroll 事件触发的函数，每隔一定时间才会执行一次 scorll 事件触发的函数**。

### 节流 throttle 代码（定时器）：

```javascript
window.onload = function () {
  //获取按钮并绑定事件
  var myThrottle = document.getElementById('throttle');
  myThrottle.addEventListener('click', throttle(sayThrottle));
};
//节流函数
function throttle(fn) {
  //通过闭包保存一个标记
  let canRun = true;
  return function () {
    //在函数开头判断标志是否为true，不为true则中断函数
    if (!canRun) {
      return;
    }
    //将canRun设置为false，防止执行之前再被执行
    canRun = false;
    //定时器
    setTimeout(() => {
      fn.call(this, arguments);
      //执行完事件(例如调用完接口)之后，重新将这个标志设true
      canRun = true;
    }, 1000);
  };
}
//需要节流的事件
function sayThrottle() {
  // ...有些需要防抖的工作，在这里进行
  console.log('节流成功~');
}
```

## 其他应用场景

讲完了这两个技巧，下面介绍一下平时开发中常遇到的场景：

- 搜索框 input 事件，例如要支持输入实时搜索可以使用节流方案（间隔一段时间就必须查询相关内容），或者实现输入间隔大于某个值（如 500ms），就当做用户输入完成，然后开始搜索，具体使用哪种方案要看业务需求。
- 页面 resize 事件，常见于需要做页面适配的时候。需要根据最终呈现的页面情况进行 dom 渲染（这种情形一般是使用防抖，因为只需要判断最后一次的变化情况）

参考文章：[浅谈 js 的防抖和节流](https://segmentfault.com/a/1190000018428170)
