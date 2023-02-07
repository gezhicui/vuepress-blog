---
title: vue-router的实现
date: 2023-01-02 20:46:50
tags:
  - vue-router
categories:
  - 框架
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
permalink: /pages/8c0e5b/
---

源码地址: [实现一个 vue-router，包含 router 内部的各种实现，并在 vue 项目中正常使用](https://github.com/gezhicui/mini-vue-router)

## 前端路由核心原理

**hash**

- `#` 号后面的就是 `hash` 的内容，本质上是借用锚点
- 可以通过 `location.hash` 拿到
- 可以通过 `onhashchange` 监听 `hash` 的改变

**history**

- `history` 即正常路径
- 可以通过 `location.pathname` 拿到
- 可以用 `onpopState` 监听 `history` 的变化

<!-- more -->

## 目录结构(src/vue-router)

`src\router.js` 中从我定义的 `vue-router文件夹`导入 `vue-router`

> import Router from '@/vue-router'

```
├─vue-router (vue-router源码文件)
│  ├─components (存放link view这两个组件的实现)
|  │  ├─link.js (router-link 组件实现)
|  │  ├─view.js (router-view 组件实现)
│  ├─history (路由的核心实现)
|  │  ├─base.js (路由基类实现，存放路由使用中通用核心方法)
|  │  ├─hash.js (HashHistory实现，继承于base中的History,存放hash路由特定方法和监听事件)
|  │  ├─html5.js (HTML5History实现，继承于base中的History,存放history路由特定方法和监听事件)
│  ├─create-matcher.js (实现`match`,`addRoutes`方法的文件)
│  ├─create-router-map.js (实现用户传入的路由配置扁平化文件)
│  ├─index.js (主入口文件 包含install的实现和VueRouter类的实现)
```

## Vue-Router 执行顺序

### 初始化执行

1、`main.js`作为项目的入口文件，导入了`src/router.js`

2、在`src/router.js`中，做了三件事:

- 导入 `vue-router`，用来创建他的实例

- 获取到`Vue`的构造函数，通过`Vue.use` 执行 `install` 方法，事先为组件实例`mixin`了一个在`beforeCreate`生命周期中进行处理的方法

- 生成路由实例，扁平化处理用户传入的路由配置项 , 提供 `match` 方法对扁平化路径进行查找，同时，根据 `mode` 去创建路由对象，初始化 `current(当前路由)`,初始化核心方法

3、`main.js`中获取到路由实例，把路由实例传入 `Vue` 的构造函数中处理

4、`new Vue` 时创建组件实例， `Vue` 为每个组件实例递归挂载了 `beforeCreate` 生命周期，但是我们之前`mixin`了一个`beforeCreate`,所以每个组件都会执行我们在该生命周期中`mixin`的处理函数

5、在`mixin`的处理函数中，在 `Vue` 递归创建实例时，会判断当前的组件实例，找到根实例，向根实例添加`_routerRoot`(Vue 根实例)，`_router`(实例化的路由对象)

6、执行路由实例中的 `init` 方法

- `init` 接收 `Vue` 根实例作为参数

- 获取到 `new VueRouter` 时创建的路由实例对象

- 执行路由实例对象的核心方法 `**transitionTo**`,

```
`transitionTo`中执行内容：

1、执行match方法搜索路由匹配项

2、改变current为匹配的路由项

3、回调listen方法，该方法内容是拿到Vue 根实例，修改Vue根实例中的_route为current，触发响应式

4、监听路由改变，路由改变就重新调用transitionTo方法
```

7、 向 `listen` 方法传入 `Vue` 根实例,给 `transitionTo` 作为回调

8、把`_route` 设置成响应式对象，值为 `current`，我们之前做了监听路由改变的操作，当路由改变时重新调用 `transitionTo` 方法,修改 `current`，触发响应式操作。

### 各组件中

由于 `router-view` 内部用到了 `current`这个变量，变量变化时会触发响应式，会刷新组件。

#### router-view 实现原理

采用函数式组件实现

- `render` 函数中传入组件的上下文 `context`, 它可以获取到当前 `router-view` 的父组件(即 `router-view` 所在的组件),上下文对象打印出来如下：

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/QQ%E5%9B%BE%E7%89%8720230207160226.png)

- 获取 `matched ` ,`matched ` 中的内容如下：

```
例如路径是/about/a
matched={
  path:/about/a,
  matched:[/about组件记录,/about/a组件记录]
}
```

- 设置 `depth = 0`

- 接下来就是递归操作，如下

1、设当前路径是`/about/a` ，则有两个组件需要渲染，一个是`/about` 的组件，一个是`/a` 的组件,[/about 组件，/a 组件]

2、首先渲染`/about`，vue 向上找，直到找到根元素，发现这是第一个 `router-view`，就渲染 `matched[0]`的组件

3、其次渲染`/a`,vue 向上找，直到找到根元素，发现`/about` 中有一个 `router-view`，`depth++`,就渲染 `matched[1]`的组件

4、 以此类推，`depth` 的值即为组件深度

#### router-link 实现原理

采用函数式组件实现,没啥好说的，非常简单，看源码就行

大功告成！！！！这里我们就实现了 Vue-router 的核心逻辑

## 踩坑记录！！

### popState 无法监听 pushState 和 replaceState 事件

我一直以为`pushState`和`replaceState`可以触发`popState`事件，但是在监听`history`路由变化时，发现怎么都监听不到路由变化事件，经过各种百度之后才知道：

实际的情况是 `history.pushState/replaceState` 并**不会触发 onpopstate 事件**，onpopstate **只有用户点击浏览器后退和前进按钮时，或者使用 js 调用 back、forward、go 方法时才会触发**。那么我们如何监听 history.pushState/replaceState ？

答案是：**重写 + 自定义事件**

重写 `history.pushState/replaceState` 使其在执行后触发一个自定义事件，我们通过监听这个自定义事件来接收视图变化通知，上代码：

```js
const bindEventListener = function (type) {
  const historyEvent = history[type];
  return function () {
    const newEvent = historyEvent.apply(this, arguments);
    const e = new Event(type);
    e.arguments = arguments;
    window.dispatchEvent(e);
    return newEvent;
  };
};
history.pushState = bindEventListener('pushState');
history.replaceState = bindEventListener('replaceState');
```

这样就创建了 2 个全新的事件，事件名为 pushState 和 replaceState，我们就可以在全局监听：

```js
window.addEventListener('replaceState', function (e) {
  console.log('THEY DID IT AGAIN! replaceState 111111');
});
window.addEventListener('pushState', function (e) {
  console.log('THEY DID IT AGAIN! pushState 2222222');
});
```

这样，我们在调用`history.pushState/replaceState`事件时，就能触发`window`上自定义的`pushState/replaceState`事件，这个事件中存放我们自定义的处理函数，处理函数与`popState`中的处理函数操作保持一致，这样就能实现`history.pushState/replaceState`导致的`history栈`变化时，触发和`popState`相同的处理操作
