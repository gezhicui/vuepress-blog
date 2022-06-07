---
title: TypeScript中的new()
date: 2021-03-12 21:48:43
tags:
  - TypeScript
categories:
  - JavaScript
permalink: /pages/29b460/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

什么是 Typescript 中的 new()?

我在有关泛型的正式文档中遇到了 new()。

这是代码上下文：

```ts
function create<T>(c: { new (): T }): T {
  return new c()
}
function a:create
```

上面的代码已转换为以下 JavaScript 代码：

```ts
function create(c) {
  return new c();
}
```

看到这里，我不太明白 new()是 JavaScript 中的非法语法。在 TypeScript 中是什么意思？

此外，{new()：T;是什么？} 意思是？我知道它一定是类型，但是如何？

其实 new()在打字稿中描述了构造函数签名。这意味着它描述了构造函数的形状。例如{new():T; }。它是一种类型。这是类的类型，其构造函数不接受任何参数。来看以下示例

```ts
function create<T>(c: { new (): T }): T {
  return new c();
}
```

这意味着函数 create 接受一个参数，其构造函数不接受任何参数，并返回**类型 T 的实例**。

```ts
function create<T>(c: { new (a: number): T }): T;
```

这意味着 create 函数接受一个参数，该参数的构造函数接受一个数字 a 并返回类型 T 的实例。另一种解释方式可以是以下类的类型

```ts
class Test {
  constructor(a: number) {}
}
```

以上代码将是`{new(a：number):Test}`

来看个示例：

```ts
class Animal {
  numLegs: number = 1;
}
class Lion extends Animal {
  keeper: string = 'aa';
}
function createInstance<A extends Animal>(c: new () => A): A {
  return new c();
}
console.log(createInstance(Lion).keeper); // aa
console.log(createInstance(Lion).numLegs); //1
```
