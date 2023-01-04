---
title: Object.create()和new的区别
date: 2022-03-10 10:11:18
tags:
  - JavaScript
categories:
  - JavaScript
permalink: /pages/e904d9/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

js 中创建对象的方式一般有两种：`Object.create`和`new`

```js
const Base = function () {};
const o1 = Object.create(Base);
const o2 = new Base();
```

在讲述两者区别之前，我们需要知道：

- 构造函数 `Foo` 的原型属性 `Foo.prototype` 指向了原型对象。
- 原型对象保存着实例共享的方法，有一个指针 `constructor` 指回构造函数。
- js 中只有函数有 `prototype` 属性，所有的对象只有`__proto__`隐式属性。

好了，首先，我们来看看 **var o1 = new Base()**的时候 new 做了什么。

```js
//创建一个空对象o1
var o1 = new Object();
//o1的_proto_指向要实例化的构造函数的prototype
o1._proto_ = Base.prototype;
//改变this指向
Base.call(o1);
```

new 做的操作就是先创建一个新的对象 o1，这时 o1 的`__proto__`指向 `Object.prototype`。然后更改 o1 的`__proto__`指向 `Base.prototype`。最后用 call 强行转换作用环境，将构造函数的 this 指向 o1，也就是 o1 拥有了构造函数 Base 定义的全部属性。

**再看看 Object.create 的实现方式**

```js
Object.create = function (Base) {
  var F = function () {};
  F.prototype = Base;
  return new F();
};
```

首先创建一个空函数 F，函数的 `prototype` 指向 `Base` 函数。new 一个函数的实例，即让该实例的`__proto__`指向函数 `F` 的 `prototype`，也就是 `Base` 函数，最后将该实例返回。

即通过 `Object.create` 创建的对象 `o2`，实际上完成了 `o2._proto_ = Base` 的操作(重点在这，如果是 new 创建的对象, o2._proto_ = Base.prototype)。注意传入的参数 Base2 是一个对象。如果传入的是一个构造函数的话，该实例是无法继承的。

来看个简单例子

```js
//定义一个对象
var person = {
  name: 'Nicholas',
  friends: ['John', 'Jane'], // 引用类型值属性共享
};
//用Object来代替Object.create实现相同功能
ObjectCreate = function (Base) {
  //创建一个空构造函数F
  var F = function () {};
  //构造函数的prototype指向Base
  F.prototype = Base;
  //返回F的实例
  return new F();
};
var onePerson = ObjectCreate(person);

console.log(onePerson.name); // "Nicholas",
```

由于`F.prototype = Base`，所以实例化出来的 onePerson.name 会去 F.prototype.name,即 Base.name 上找
