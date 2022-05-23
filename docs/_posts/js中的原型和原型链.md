---
title: js中的原型和原型链
date: 2020-06-16 17:05:57
tags:
  - 前端
  - 原型
  - JavaScript
categories:
  - 前端
  - JavaScript
permalink: /pages/db0739/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

在 ES6 之前，面向对象是通过构造函数来实现的。构造函数的方法很好用，但是存在一个**浪费内存**的问题。

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/gouzaohanshu.png)

如图，每创建一个对象，都要开辟一个新的内存区域，**我们希望所有的对象使用同一个函数，这样就比较节省内存，那么我们要怎样做呢？**

## 构造函数原型 prototype

&emsp;&emsp;构造函数通过原型分配的函数是所有对象所**共享的**。
&emsp;&emsp;JavaScript 规定，每一个构造函数都有一个`prototype`属性，指向另一个对象。注意这个`prototype`就是一个对象，这个对象的所有属性和方法，都会被构造函数所拥有。称为**_原型对象_**

&emsp;&emsp;我们可以把那些不变的方法，直接定义在 prototype 对象上，这样所有对象的实例就可以共享这些方法。这就是原型的作用

&emsp;&emsp;举个栗子

```javascript
//解决构造函数的问题
function star(uname, age) {
  this.uname = uname;
  this.age = age;
}
//把sing方法挂载到原型上
star.prototype.sing = function () {
  console.log('我会唱歌');
};

var ldh = new star('刘德华', 18);
var zxy = new star('张学友', 18);
ldh.sing();
zxy.sing();
```

---

## 对象原型 **proto**

&emsp;&emsp;对象都会有一个属性 **proto** 指向构造数的 prototype 原型对象，之 以我们对可以使用构造函数 prototype 原型对象的性和方法，就是因为对象 **proto**原型的存在。

```javascript
console.log(ldh.__proto__ === star.prototype);
//输出结果为true
//方法的查找规则：首先先看1dh对象身上是否有sing方法，如果有就执行这个对象上的sing
//如果么有sing这个方法，因为有_proto的存在，就去构造函数原型对象prototype身上去查找sing这个方法I
```

&emsp;&emsp;proto 对象原型的意义就在于为对象的查找机制提供一个方向，或者说一条路线，但是它是一个非标准属性，因此实际开发中，不可以使用这个属性，它只是内部指向原型对象 prototype
![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/proto.png)

---

## constructor 构造函数

&emsp;&emsp;对象原型（ _proto_ ）和构造函数（prototype）原型对象里面都有一个属性 constructor 属性，constructor 我们称为构造函数，因为它指回构造函数本身。

```javascript
console.log(star.prototype.constructor);
console.log(ldh.__proto__.constructor);
//输出的都是star这个构造函数
```

当我们想在 prototype 上添加多个方法的时候：

```javascript
star.prototype = {
  sing: function () {
    console.log('我会唱歌');
  },
  movie: function () {
    console.log('我会演电影');
  },
};
```

**这个时候，protptype 被完全覆盖掉了**
我们就要添加一个语句让 prototype 重新指回构造函数

```javascript
constructor: star;
//如果我们修改了原来的原型对象，给原型对象赋值的是一个对象，则必须手动的利用constructor指回原来的构造函数
```

---

## 构造函数、实例、原型对象三者之间的关系

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/构造函数原型原型对象关系.png)

---

## 原型链

&emsp;&emsp;**只要是对象就有\_proto 原型，指向原型对象**

一张图看懂原型链：
![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/原型链.png)

- 我们 star 原型对象里面的*proto*原型指向的是 Object.prototype

- 我们 object.prototype 原型对象里面的*proto*原型指向为 null

---

## JavaScript 的成员查找机制（规则）

- 当访问一个对象的属性（包括方法）时，首先找这个**对象自身**有没有该属性。

- 如果没有就查找它的原型（也就是*proto*指向的（**prototype 原型对象**）。

- 如果还没有就查找原型对象的原型（**Object 的原型对象**）。
- 依此类推一直找到 Object 为止（**null**）。

---

## 原型对象中的 this 指向问题

```js
star.protptype.sing = function () {
  console.log('sing');
  that = this;
};
var ldh = new star('刘德华', 18);
//在构造函数中，里面this指向的是对象实例ldh
ldh.sing();
console.log(that === this);
//输出结果为true
//原型对象函数里面的this 指向的是实例对象ldh
```
