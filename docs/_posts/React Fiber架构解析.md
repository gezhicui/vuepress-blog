---
title: React Fiber架构解析
date: 2022-08-25 09:39:43
tags:
  - React
categories:
  - 框架
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
permalink: /pages/2e6d49/
---

公司的技术栈是 `react`，但是其实使用了这么久的 `react`，对于 `react` 的实现原理还是两眼一抹黑，于是一个月来都在通过各种文章视频来了 `react` 的底层实现，整理总结出 **`react fiber`** 架构的具体实现,要讲清 `react fiber` 的架构还是挺不容易的，个人认为 `react fiber` 架构比较复杂，所以本篇文章应该要持续更新一段时间

源码地址：[https://github.com/gezhicui/mini-react16](https://github.com/gezhicui/mini-react16)

<!-- more -->

## createElement

在执行 react 的整个流程前,首先要解析一下`jsx`,在 react 项目的开发中，通常要引入 react：

```js
import React from 'React';
```

但是，我们并没有用到`React`上的什么方法，但是为啥要引入呢??原因就是因为`jsx`在`babel`编译过后会变成`React.createElement`，引入`React`，主要是预防找不到`React`而产生报错。所以，实现`React`要从先实现`createElement`开始

我们来看一下这样一段代码：

```js
const title = <h1 className="title">Hello, world!</h1>;
```

这段代码并不是合法的 js 代码，它是`jsx`的语法扩展，通过它我们就可以**很方便的在 js 代码中书写 html 片段**。

本质上，**jsx 是语法糖**，上面这段代码会被`babel`转换成如下代码：

```js
const title = React.createElement(
  'h1',
  { className: 'title' },
  'Hello, world!'
);
```

本质上来说 JSX 是`React.createElement(component, props, ...children)`方法的语法糖。

当我们有如下的 jsx 结构时：

```js
let element1 = (
  <div id="A1" style={style}>
    A1文本
    <div id="B1" style={style}>
      B1文本
      <div id="C1" style={style}>
        C1文本
      </div>
      <div id="C2" style={style}>
        C2文本
      </div>
    </div>
    <div id="B2" style={style}>
      B2文本
    </div>
  </div>
);
```

通过`babel`的转换，会变成这样：

```js
React.createElement(
  'div',
  { id: 'A1' },
  React.createElement(
    'div',
    { id: 'B1' },
    'B1文本',
    React.createElement('div', { id: 'C1' }, 'C1文本'),
    React.createElement('div', { id: 'C2' }, 'C2文本')
  ),
  React.createElement('div', {
    id: 'B2',
  })
);
```

那么，现在就来自己实现一下`createElement`处理方法

新建`react.js`文件，编写方法

```js
/**
 * 创建元素（虚拟DOM）的方法
 * @param {*} type  元素的类型 div span p
 * @param {*} config 配置对象 属性 key ref
 * @param  {...any} children 所有的儿子，传入一个数组,如果没有儿子，如<div/>，则为空数组
 */
function createElement(type, config, ...children) {
  return {
    type,
    props: {
      ...config, //属性扩展 id、key、style
      children: children.map((child) => {
        //兼容处理，如果是普通元素返回自己，如果是文本类型，返回文本元素对象
        //比方说B1文本那么children就是["B1文本"]，改为了
        //{type:Symbol(ELEMENT_TEXT),props:{text:"B1文本",children:[]}}也不可能有children了
        return typeof child === 'object'
          ? child
          : {
              type: 'ELEMENT_TEXT',
              props: { text: child, children: [] },
            };
      }),
    },
  };
}
const React = {
  createElement,
};
export default React;
```

完成了`createElement`,在`index.js`文件中导入,并查看`jsx`处理完的结果

```js
import React from './react';

let style = { border: '3px solid red', margin: '5px' };
let element1 = (
  <div id="A1" style={style}>
    A1文本
    <div id="B1" style={style}>
      B1文本
      <div id="C1" style={style}>
        C1文本
      </div>
      <div id="C2" style={style}>
        C2文本
      </div>
    </div>
    <div id="B2" style={style}></div>
  </div>
);
console.log(element1);
```

打印结果如下，由于层级过多，有的就不展开了：

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220825163253.png)

这样，完整的节点`jsx`对象结构就生成出来了，**这就是`react`的`virtual-dom`**

## ReactDOM.render

有了虚拟 dom，就可以使用`ReactDOM.render`方法把虚拟 dom 挂载到页面上了，该方法创建了一个根节点(`rootFiber`)，把刚刚解析过的虚拟 dom 挂载在该节点下

```js
// 新建react-dom.js

/**
 * render是要把一个元素渲染到一个容器内部
 * @param {*} element 元素
 * @param {*} container 容器
 */
function render(element, container) {
  let rootFiber = {
    tag: TAG_ROOT, //每个virtual-dom会有一个tag标示此元素类型
    stateNode: container,
    props: { children: [element] },
  };
  scheduleRoot(rootFiber);
}

const ReactDOM = {
  render,
};
export default ReactDOM;

//index.js中
import ReactDOM from './react-dom';

ReactDOM.render(element1, document.querySelector('#root'));
```

可以看到，`ReactDOM.render`中构建了一个 root 节点，把刚刚的 `virtual-dom` 挂在这个 root 节点下，然后调用`scheduleRoot`进行调度，fiber 就是在`scheduleRoot`调度的过程中产生的

既然本文是讨论 fiber,那什么是 fiber??

## Fiber

在了解什么是 fiber 之前，我们应该要了解一下为什么会出现 fiber

### React 15 架构

每当有更新发生时，`Reconciler(协调器)`会做如下工作：

- 调用函数组件、或 class 组件的 render 方法，将返回的**JSX 转化为虚拟 DOM**
- 将虚拟 DOM 和上次更新时的虚拟 DOM **对比**
- 通过对比**找出本次更新中变化的虚拟 DOM**
- 通知 `Renderer` 将变化的虚拟 DOM 渲染到页面上

对于 React 的更新来说，递归遍历应用的所有节点由于递归执行，计算出差异，然后再更新 UI。**递归是不能被打断的**，所以更新一旦开始，中途就无法中断。当层级很深时，递归更新时间超过了 16ms(小于 60 帧)，用户交互就会卡顿。

### React 16 的设计思想

React 16 实现的思路是这样的：**将运算切割为多个步骤，分批完成**。在完成一部分任务之后，将控制权交回给浏览器，让浏览器有时间进行页面的渲染。等浏览器忙完之后，再继续之前未完成的任务。这就是 React 16 中的 Fiber 设计思想。

为了达到能中断处理这种效果，就需要有一个调度器 `(Scheduler)` 来进行任务分配，`Fiber Reconciler` 在执行过程中，会分为 2 个阶段:

- 1、阶段一，**生成 Fiber** 树，得出需要更新的节点信息。这一步是一个渐进的过程，**可以被打断**。
- 2、阶段二，将需要更新的节点**一次性批量更新**，这个过程**不能被打断**。

阶段一可被打断的特性，让优先级更高的任务先执行，从框架层面大大降低了页面掉帧的概率。

### Fiber 的实现细节

`Fiber Reconciler` 在阶段一进行 Diff 计算的时候，会生成一棵 Fiber 树。这棵树是在 Virtual DOM 树的基础上增加额外的信息来生成的，它本质来说是一个链表。

每个 Fiber 节点有个对应的 React element，多个 Fiber 节点是如何连接形成树呢？靠如下三个属性

```js
// 指向父级Fiber节点
this.return = null;
// 指向子Fiber节点
this.child = null;
// 指向右边第一个兄弟Fiber节点
this.sibling = null;
```

举个例子，如下的组件结构：

```js
function App() {
  return (
    <div>
      i am
      <span>KaSong</span>
    </div>
  );
}
```

对应的`Fiber`结构为：

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220829173317.png)

用数据结构来描述则为

```js
const newFiber = {
  tag:null, //组件类型 CLASS、FUNCTION_COMPONENT、TEXT、HOST等
  type:null  //createElement时接收的节点类型type,如果是原生的节点类型，那么就是一个字符串(div、span等),如果是一个组件(我们自己定义的、或者React内置提供的)那么就是一个变量
  props: null, //虚拟dom属性，如style
  stateNode: null, //如果是原生的节点类型，是虚拟dom对应的真实节点，如果是class/func组件，则是组件实例
  updateQueue: null, // 数据更新队列
  return: null, //父fiber
  alternate: null, //捞fiber，用于新旧fiber对比更新
  effectTag: null, //副作用标示，render会收集副作用 增加 删除 更新
  nextEffect: null, //fiber节点组合成effectlist链表，指向链表当前节点的下一个节点
};
```

### 如何实现浏览器控制权切换？

react 16 的可中断调度，其实就是基于浏览器的 `requestIdleCallback`这个 api，在**每个帧的空闲时间**来处理`fiber`,

每个帧的开头包括样式计算、布局和绘制，JavaScript 执行, Javascript 引擎和页面渲染引擎在同一个渲染线程,**GUI 渲染和 Javascript 执行两者是互斥的**，如果某个任务执行时间过长，浏览器会推迟渲染。

假如某一帧里面要执行的任务不多，在不到 16ms（1000ms/60fps)的时间内就完成了上述任务的话，那么这一帧就会有一定的空闲时间，这段时间就恰好可以用来执行`requestIdleCallback`的回调

由于`requestIdleCallback`利用的是帧的空闲时间，所以就有可能出现浏览器一直处于繁忙状态，导致回调一直无法执行，这其实也并不是我们期望的结果，那么这种情况我们就需要在调用`requestIdleCallback`的时候传入第二个配置参数`timeout了`

```js
// timeout指定多少毫秒之后若浏览器还是没空，则不等了，强制执行
requestIdleCallback(workFunction, { timeout: 2000 });
```

知道了`requestIdleCallback`的用法，就可以来看一段示例：

```js
/**
 * 函数fn接受deadline对象作为参数，deadline对象有两个属性：timeRemaining和didTimeout。
 * timeRemaining() 返回当前帧还剩余的毫秒数。
 * didTimeout 指定的时间是否过期。
 */
function workFunction(deadline) {
  while (
    (deadline.timeRemaining() > 0 || deadline.didTimeout) &&
    taskList.length > 0
  ) {
    //如果当前帧还有剩余时间，或超时了但是任务还没执行完，就执行任务
    doWork();
  }
  // 如果当前帧没有剩余时间了，就把控制权还给浏览器,在下个帧继续执行任务
  if (taskList.length > 0) {
    requestIdleCallback(workFunction);
  }
}

// 执行requestIdleCallback，workFunction是要执行的函数
requestIdleCallback(workFunction, 2000);
```

我们可以看到，`taskList`就是待执行的任务列表，而`fiber`调度的实现就是把`fiber`变成一个链表，按链表顺序执行，时间不够就把链表的下个节点保存等到下一帧执行

## schedule 过程发生了什么？

了解完 fiber，接下来就来完成 fiber 的实现

react 从根节点开始渲染和调度，分为两个阶段 : **diff+render 阶段，commit 阶段**

### diff+render 阶段

diff+render 阶段 对比新旧虚拟 DOM，进行增量更新或创建,花时间长，**可进行任务拆分，此阶段可暂停**

render 阶段两个任务：

- 1.根据虚拟 DOM 生成 fiber 树
- 2.收集 effectlist

render 阶段的成果是 `effectlist`， 即知道哪些节点更新哪些节点增加删除了

### commit 阶段

commit 阶段，根据 render+diff 阶段的成果（effectlist），进行 DOM 更新创建，**此阶段不能暂停**

下面通过代码看看`schedule`过程，整个过程从`react-dom.js`文件中的`scheduleRoot`方法开始

## schedule 过程代码实现

### scheduleRoot

在`scheduleRoot`中，把`rootFiber`保存在两个变量中，`workInProgressRoot`保存 RootFiber 应用的根，`nextUnitOfWork`保存供`requestIdleCallback`消费的下个任务单元

```js
let workInProgressRoot = null; //RootFiber应用的根
let nextUnitOfWork = null; //下一个工作单元

export function scheduleRoot(rootFiber) {
  // rootFiber保存在workInProgressRoot和nextUnitOfWork中
  workInProgressRoot = rootFiber;
  nextUnitOfWork = rootFiber;
}
```

在`workInProgressRoot`和`nextUnitOfWork`赋值之后，`requestIdleCallback`就开始工作了

### workLoop

```js
/**
 * 回调返回浏览器空闲时间，判断是否继续执行任务
 * @param {*} deadline
 */
function workLoop(deadline) {
  //react是否要让出控制权
  let shouldYield = false;
  // 如果当前有下个工作单元，且不需要让出控制权，则处理下个工作单元
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    // 如果当前帧剩余时间过短，则让出控制权
    shouldYield = deadline.timeRemaining() < 1;
  }
  //如果全部工作单元处理结束，则进入commit
  if (!nextUnitOfWork && workInProgressRoot) {
    console.log('render阶段结束');
    commitRoot();
  }
  //每一帧都要执行这个代码
  window.requestIdleCallback(workLoop, { timeout: 500 });
}

//react询问浏览器是否空闲,这里有个优先级的概念 expirationTime
window.requestIdleCallback(workLoop, { timeout: 500 });
```

在上面的代码中，浏览器会在每一帧的空闲时间执行`workLoop`,在`workLoop`方法中,只要存在下一个任务单元，且当前帧有空闲时间，则开始工作，所以在`workInProgressRoot`和`nextUnitOfWork`赋值之后，浏览器就会在每帧的空闲时间进行 fiber 的处理了，在代码中我们也可以发现，处理 fiber 的函数是`performUnitOfWork`,接下来看看这个处理函数做了什么

### performUnitOfWork

```js
// 每一帧需要穿插执行的内容
function performUnitOfWork(currentFiber) {
  beginWork(currentFiber);
  if (currentFiber.child) {
    return currentFiber.child; //有孩子返回孩子
  }
  // 没孩子
  while (currentFiber) {
    //完成当前节点
    completeUnitOfWork(currentFiber);
    console.log(currentFiber);
    //有弟弟返回弟弟
    if (currentFiber.sibling) {
      return currentFiber.sibling; //有弟弟返回弟弟
    }
    //没弟弟找到父亲继续执行
    currentFiber = currentFiber.return;
  }
}
```

以上部分代码现在看起来有点懵，因为现在能知道的信息就是`child`属性是在虚拟 dom 上的，代表当前节点的子节点们，但是`sibling`和`return`是啥？别急，我们先从前几行开始看

```js
beginWork(currentFiber);
if (currentFiber.child) {
  return currentFiber.child; //有孩子返回孩子
}
```

这部分应该目前还看得懂，`beginWork`方法处理当前虚拟 dom，处理完后如果当前虚拟 dom 有子节点，则让`nextUnitOfWork`等于子节点，把子节点变成下个任务单元供`workLoop`继续进行可中断调度

那`beginWork`方法中做了什么？

### beginWork

```js
function beginWork(currentFiber) {
  if (currentFiber.tag === TAG_ROOT) {
    //根节点
    updateHostRoot(currentFiber);
  } else if (currentFiber.tag === TAG_TEXT) {
    //文本节点
    updateHostText(currentFiber);
  } else if (currentFiber.tag === TAG_HOST) {
    //原生dom节点
    updateHost(currentFiber);
  }
  // console.log('beginWork', currentFiber);
}

function updateHostRoot(currentFiber) {
  //先处理自己 如果是一个原生节点，创建真实DOM 2.创建子fiber
  let newChildren = currentFiber.props.children; //[element]
  reconcileChildren(currentFiber, newChildren); //reconcile协调
}

function updateHostText(currentFiber) {
  if (!currentFiber.stateNode) {
    //如果此fiber没有创建DOM节点
    currentFiber.stateNode = createDom(currentFiber);
  }
}

function updateHost(currentFiber) {
  if (!currentFiber.stateNode) {
    //如果此fiber没有创建DOM节点
    currentFiber.stateNode = createDom(currentFiber);
  }
  const newChildren = currentFiber.props.children;
  reconcileChildren(currentFiber, newChildren);
}

function createDom(currentFiber) {
  //文本节点
  if (currentFiber.tag === TAG_TEXT) {
    return document.createTextNode(currentFiber.props.text);
  } else if (currentFiber.tag === TAG_HOST) {
    // 其他原生dom节点 如div span
    let stateNode = document.createElement(currentFiber.type);
    //处理属性
    setProps(stateNode, {}, currentFiber.props);
    return stateNode;
  }
}
```

`beginWork`就是根据每个节点的 `tag` （tag 是在根据虚拟节点创建 fiber,即后面要讲的`reconcileChildren`方法中生成的）进行相应的处理,生成真实的 dom 节点，并把节点的属性通过`setProps`方法添加在真实节点上（`setProps`的具体实现看源码），然后把真实节点挂在当前节点的`stateNode`属性上

由于根节点和原生 dom 节点会有子节点，所以要继续进行下一步`reconcileChildren`,传入当前节点的子节点，处理子节点，**生成子节点的 fiber**，而文本节点不会有子节点，所以不需要`reconcileChildren`处理子节点

### reconcileChildren

`reconcileChildren`是虚拟 dom 生成 fiber 的步骤，是 fiber 的核心之一，**作用是生成当前正在处理的节点的子节点的 fiber**，因为这个方法只用来处理子节点，这也是为什么最外层要挂一个 rootFiber 的原因。

直接看代码

```js
function reconcileChildren(currentFiber, newChildren) {
  let newChildIndex = 0; //新子节点的索引
  let prevSibiling; //上一个新的子fiber
  //遍历我们子虚拟DOM元素数组，为每一个虚拟DOM创建子Fiber
  while (newChildIndex < newChildren.length) {
    let newChild = newChildren[newChildIndex]; //取出虚拟DOM节点
    let tag;
    if (newChild.type == ELEMENT_TEXT) {
      tag = TAG_TEXT;
    } else if (typeof newChild.type === 'string') {
      tag = TAG_HOST; //如果type是字符串，那么这是一个原生DOM节点div
    }
    // 处理当前fiber的子fuber
    let newFiber = {
      tag,
      type: newChild.type,
      props: newChild.props,
      stateNode: null, //div还没有创建DOM元素
      return: currentFiber, //父Fiber returnFiber
      effectTag: PLACEMENT, //副作用标示，render会收集副作用 增加 删除 更新 第一次进来都是增加
      nextEffect: null, //effect list也是一个单链表 顺序和完成顺序一样 节点可能会少 只放需要变动的fiber节点,其他的绕过
    };
    if (newFiber) {
      if (newChildIndex == 0) {
        //如果索引是0，就是大儿子
        currentFiber.child = newFiber;
      } else {
        prevSibiling.sibling = newFiber; //大儿子指向弟弟
      }
      prevSibiling = newFiber;
    }

    newChildIndex++;
  }
}
```

`reconcileChildren`接收两个参数，第一个是当前正在处理的虚拟 dom，第二个是当前 dom 的所有子节点，该方法用于生成当前虚拟 dom 的所有子节点的 fiber，子 fiber 通过`sibling`连接，父子 fiber 通过`child`和`return`连接，这样父子组件就可以形成一个链表，方便用于中断调度

具体内容看注释就行，关键的是创建完 fiber 后把子节点串起来的代码要细细解读下

```js
if (newFiber) {
  if (newChildIndex == 0) {
    //如果索引是0，就是大儿子
    currentFiber.child = newFiber;
  } else {
    prevSibiling.sibling = newFiber; //大儿子指向弟弟
  }
  prevSibiling = newFiber;
}
```

这部分代码的意思是，在循环创建当前节点的子节点 fiber 的过程中进行以下分支操作：

- 如果是第一个子节点，则当前`fiber.child`等于第一个子节点
- 如果不是第一个子节点，则把上一个子节点的`sibling`指向自己

这样，父节点和所有子节点就通过`child`和`sibling`以及生成 fiber 过程中的`return`串起来了
