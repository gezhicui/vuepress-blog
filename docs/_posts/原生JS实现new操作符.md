---
title: 原生js实现new操作符
date: 2021-04-09 16:05:22
tags:
  - JavaScript
categories:
  - JavaScript
permalink: /pages/965179/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

new 本质上是一个语法糖，简化了我们实例化对象的步骤，今天突然比较好奇 new 的实现原理，现在我们来扒开糖衣看看 new 的本质

## new 的实现过程

在 js 中 var a = new A() 后，有这么几步：

- 1.创建一个空对象 obj{}
- 2.将 a 的 this 指向 obj{}
- 3.将 a 的*proto*指向 A 的 prototype
- 4.执行 A 的构造函数
- 5.返回 obj

## 代码实现

```js
function Test(name) {
  this.name = name;
  this.test = 'test';
}

Test.prototype.test2 = function () {
  console.log('测试原型链上的方法!');
};

function _new() {
  // 新建一个对象作为实例对象
  let newObj = {};
  // 取出传入的Test构造函数，赋值给Constructor
  let Constructor = Array.prototype.shift.call(arguments);
  // 新建对象的__proto__指向构造函数的prototype
  newObj.__proto__ = Constructor.prototype;
  // 传入剩余的参数，this的指向改为新的对象
  Constructor.apply(newObj, arguments);
  // 返回该对象
  return newObj;
}
// 测试代码
var t = _new(Test, 'yz');

t.test2(); // 测试原型链上的方法!
console.log(t.name); // name
```

详细过程都在注释中了，扒完收工！
