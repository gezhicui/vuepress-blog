---
title: 实现一个vue文件解析器
date: 2022-06-22 22:16:35
permalink: /pages/913496/
sidebar: auto
categories:
  - 框架
tags:
  - vue
  - vue-loader
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

# 如从 0 开始处理一个 vue 文件并实现简单的响应式？

在现在的前端工程化中,打包工具是不可或缺的，其中`webpack`无疑是占据了主导地位，当然也有尤大搞的`vite`,但是论生态和使用人数，至少在目前`webpack`还是更胜一筹。

打包工具能帮助我们打包前端文件，在`webpack`中，不同后缀的文件通过不同`loader`来处理。

本文就讨论下怎么实现一个处理`.vue`文件的`loader`，以及用`loader`处理完`.vue`文件怎么把内容渲染在浏览器上，并实现简单的响应式。

源码地址 [gezhicui/vue-webpack](https://github.com/gezhicui/vue-webpack)

<!-- more -->

## webpack 部分

首先前端的文件都会经过 `webpack` 打包,在`webpack`中把`.vue` 文件通过 `vue-loader` 处理。

实现一个简易的`vue-loader`,通过一系列正则，最终一个`.vue `文件的内容会被包装到一个对象中

比方说我现在的.vue 文件写了下面这些内容：

```
<template>
  <div>
    <h1>{{ count + 1 }}</h1>
    <button @click="plus(1)">+</button>
  </div>
</template>

<script>
export default {
  name: 'App',
  data () {
    return {
      count: 0
    }
  },
  methods: {
    plus (num) {
      this.count += num;
    }
  }
}
</script>
```

那么经过 `vue-loader` 处理，就会变成一个对象：

```js
{
  template:
   `<div>
     <h1>{{ count + 1 }}</h1>
      <button @click="plus(1)">+</button>
  </div>`,
  name: 'App',
  data() {
    return { count: 0 }
  },
  methods: {
    plus(num) { this.count += num; },
  }
}
```

那么，引入这个组件的时候，我们就能通过`createApp`方法，把这个对象使用 `createApp` 进行处理，挂载到页面上

## createApp 实现部分

在 vue 的`main.js`文件中，我们通常会把根组件传递给`createApp`作为入参，如:

```js
import App from './App';
import { createApp } from '../modules/vue';

createApp(App).mount('#app');
```

那我们实现的重点就在于`createApp`对**vue 组件对象**的处理，以及在`createApp`的返回内容(就是 vm)中添加`mount`方法，实现处理完的节点的挂载

接下来就一步步实现`createApp`，首先，我们先来定义一个 vm，一会儿所有的属性都可以放在 vm 上,同时把`vue-loader`解析过的文件对象中的内容给解构出来

```js
function createApp(component) {
  const vm = {};
  const { template, methods, data } = component;
}
```

### template 解析

在上面经过`vur-loader`处理后，**`template`以字符串形式**被放到对象中，所以我们可以拿到 dom 元素字符串，把他转成 dom 元素

```js
/* 
  template:
   `<div>
     <h1>{{ count + 1 }}</h1>
      <button @click="plus(1)">+</button>
  </div>`,
*/
vm.$node = createNode(template);

function createNode(template) {
  const _tempNode = document.createElement('div');
  _tempNode.innerHTML = template;
  return getFirstChildNode(_tempNode);
}
```

这样，我们就拿到了 html 接下来就是对 js 的操作

### data 响应式处理

vue 的核心就在于响应式，vue2 通过`Object.defineProperty`实现响应式，我们来实现个简单的响应式处理

首先拿到`data`,为了创建多个组件时`data`不被互相影响，所以`data`是一个函数

```js
vm.$data = data();

for (let key in vm.$data) {
  Object.defineProperty(vm, key, {
    get() {
      return vm.$data[key];
    },
    set(newValue) {
      vm.$data[key] = newValue;
      // update触发节点更新，至于实现我放到后面再说
      update(vm, key);
    },
  });
}
```

这样，我们就监听了`data`中每个属性的`get`和`set`，实现了数据的响应式处理

### 初始化数据池

在上面的 **template 解析**中，我们已经拿到了`template`转换过后的节点，但是有个问题，节点的内容没有经过任何处理，如`'{{count + 1}}'`会原封不动的展示在浏览器中，我们希望的是最终展示的是 `count` 这个变量 +1 的结果，所以我们需要对双括号语法进行解析

我们先定义一个正则表达式，匹配`{{}}`中的内容，以及定义一个节点数据池

```js
// 节点数据池
const exprPool = new Map();
// 正则获取双括号中内容
const regExpr = /\{\{(.+?)\}\}/;
```

然后，从我们刚刚定义的`vm.$node`中拿到所有节点，并查看该节点是否有双括号语法，如果有的话存入节点数据池中

```js
const allNodes = $node.querySelectorAll('*');
allNodes.forEach(node => {
  // 这里获取到的textContent是原原始的没经过任何处理的节点内容，如{{count + 1}}
  const vExpression = node.textContent;
  /* exprMatched：{
      0: "{{ count + 1 }}"
      1: " count + 1 "
      groups: undefined
      index: 0
      input: "{{ count + 1 }}"
    }
    */
  const exprMatched = vExpression.match(regExpr);
  // 如果有双括号语法
  if (exprMatched) {
    const poolInfo = checkExpressionHasData($data, exprMatched[1].trim());
    // 把节点存入节点数据池
    poolInfo && exprPool.set(node, poolInfo);
  }
});

function checkExpressionHasData(data, expression) {
  for (let key in data) {
    if (expression.includes(key) && expression !== key) {
      // count + 1,返回{key:count,expression:count+1}
      return {
        key,
        expression,
      };
    } else if (expression === key) {
      // count,返回{key:count,expression:count}
      return {
        key,
        expression: key,
      };
    } else {
      return null;
    }
  }
}
```

### 初始化事件池

处理完双括号语法，我们还需要处理`@click`这样的事件语法,首先，我们创建一个事件池,再定义两个正则分别匹配函数

```js
const eventPool = new Map();

// 匹配函数名
const regStringFn = /(.+?)\((.+?)\)/;
// 匹配函数参数
const regString = /\'(.+?)\'/;
```

同样的，我们也需要遍历所有节点

```js
const allNodes = $node.querySelectorAll('*');

allNodes.forEach(node => {
  const vClickVal = node.getAttribute(`@click`);
  if (vClickVal) {
    /* 
      比如@click='plus(1)',解析完成的fnInfo就是
      fnInfo:{
        args: [1]
        methodName: "plus"
      }
      */
    const fnInfo = checkFunctionHasArgs(vClickVal);
    const handler = fnInfo
      ? //有参函数传入args
        methods[fnInfo.methodName].bind(vm, ...fnInfo.args)
      : //无参函数直接绑定
        methods[vClickVal].bind(vm);

    //存入事件池，节点为key,事件为value
    eventPool.set(node, {
      type: vClick,
      handler,
    });
    //删除dom上的attr，不然浏览器查看源代码就会显示自定义事件  这样不好
    node.removeAttribute(`@${vClick}`);
  }
});

function checkFunctionHasArgs(str) {
  const matched = str.match(regStringFn);

  if (matched) {
    const argArr = matched[2].split(',');
    const args = checkIsString(matched[2])
      ? argArr // ['1']
      : argArr.map(item => Number(item));

    return {
      methodName: matched[1],
      args,
    };
  }
}
function checkIsString(str) {
  return str.match(regString);
}
```

这样，我们有拥有了节点数据池和事件池,接下来我们就要拿节点数据池和事件池做操作了

### 绑定事件处理

有了事件池，我们就要**把事件池中的事件绑定到 dom 元素上去**，让事件能够触发。这步其实是很容易的，因为我们把 `vue` 事件加入事件池中时，**key 是 dom 元素**，**value 是事件处理函数**，只要把他们两个互相绑定就行

```js
function (vm) {
  //node:key  info:value
  for (let [node, info] of eventPool) {
        // type:事件类型  handler：事件处理函数
    let { type, handler } = info;
    //在vue中，是用this.function 去访问方法，所以方法要被绑定到vm上
    vm[handler.name] = handler;
    //给节点绑定事件处理函数
    node.addEventListener(type, vm[handler.name], false);
  }
}
```

### render 页面

执行完上面的内容，我们就到了最后一步 `render 页面`了，我们只要**把节点数据池中的节点内容渲染到浏览器**上

```js
function render(vm) {
  exprPool.forEach((info, node) => {
    _render(vm, node, info);
  });
}

function _render(vm, node, info) {
  //info:{key: 'count',expression 'count + 1'}
  const { expression } = info;
  //expression是一个字符串，为了执行字符串，所以我们需要new Function
  const r = new Function(
    'vm',
    'node',
    `
    with (vm) {
      node.textContent = ${expression};
    }
  `
  );

  r(vm, node);
}
```

在这里，我们先解决两个问题

- with 是干啥用的？
- 为什么\_render 要抽离出来？

首先先来介绍下 with

with 的作用是用来改变标识符的查找优先级，优先从 with 指定对象的属性中查找。e.g:

```js
var a = 1;
var obj = {
  a: 2,
};
with (obj) {
  console.log(a); //2
}
```

那为什么\_render 要单独抽成一个函数？ 因为在前面的 **data 响应式处理** 中，`set`被触发时，我们需要拿到新的数据值去`update`页面元素，这时候就也会用到`render`函数，那就简单实现下上面提到的`updata`

```js
export function update(vm, key) {
  //在节点数据池中查找哪个节点的key==当前改变的key，找到则重新render
  exprPool.forEach((info, node) => {
    if (info.key === key) {
      _render(vm, node, info);
    }
  });
}
```

到此为止，就能实现一个完整的不通过任何第三方插件解析 vue 文件，并实现简单的响应式处理了！！
