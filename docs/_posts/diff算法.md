---
title: Diff算法
date: 2022-09-08 09:47:29
tags:
  - React
  - Vue
  - 数据结构与算法
categories:
  - 框架
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
permalink: /pages/99dac3/
---

Vue 和 React 都用到了虚拟 dom,虚拟 dom 的核心是进行虚拟节点对比，用来存储两个节点不同的地方，最后用记录的消息去局部更新 Dom。这个过程就称为`diff`

本文记录一下**React 中的 diff 算法、Vue2 的双端 diff 算法、Vue3 的快速 diff 算法**核心思想

<!-- more -->

## 传统 diff 算法的时间复杂度 O(n ^3)

diff 算法的核心就是进行两棵树的对比更新，所以要遍历一棵树的所有节点与另一棵树的所有节点进行比较，我们可以用数组比较来模拟：

```js
const arr1 = [1, 2, 3, 4, 5, 6];
const arr2 = [2, 4, 5, 1, 8, 6];
const res = [];
for (let i = 0; i < arr1.length; i++) {
  for (let j = 0; j < arr2.length; j++) {
    if (arr1[i] === arr2[j]) {
      res.push([i, j]);
    }
  }
}
console.log(res); //[ [ 0, 3 ], [ 1, 0 ], [ 3, 1 ], [ 4, 2 ], [ 5, 5 ] ]
```

以上方法就可以找出两个数组中相同元素对应的位置，时间复杂度是`O(n^2)`

在对比过程中发现旧节点在新的树中未找到，那么就需要把旧节点删除，删除一棵树的一个节点(找到一个合适的节点放到被删除的位置)的时间复杂度为`O(n)`,同理添加新节点或修改节点的复杂度也是`O(n)`,合起来 diff 两个树的复杂度就是`O(n^3)`

如果有 1000 个节点,`O(n^3)`的执行的计算量开销将在十亿的量级范，围就会变的十分巨大，所以 Vue 和 React 把 diff 算法优化到了 `O(n)`

## dom diff 的取舍

O(n^3)和 O(n)差了两个指数级，尤大说过框架设计就是不断地舍取，在如此巨大的优化面前肯定要舍弃一些东西，所以 dom diff 并不是一棵树的所有节点与另一棵树的所有节点比较，而是做了如下优化：

**为了降低算法复杂度，diff 会预设三个限制**：

- 1.**只对同级元素进行 Diff**。Web UI 中 DOM 节点跨层级的移动操作特别少，如果一个 DOM 节点在前后两次更新中跨越了层级，那么不会尝试复用他。
- 2.**两个不同类型的元素会产生出不同的树**。如果元素由 div 变为 p，算法会销毁 div 及其子孙节点，并新建 p 及其子孙节点。
- 3.开发者可以通过 **key** 来暗示哪些子元素在不同的渲染下能保持稳定。

## React diff

下面来看看 react 中的 diff 是怎么做的

### tree diff

既然 DOM 节点跨层级的移动操作少到可以忽略不计，那么`React`通过`updateDepth` 对 `Virtual DOM 树`进行层级控制，也就是同一层在对比的过程中，如果发现节点不在了，会完全删除，不会对其他地方进行比较，这样只需要对树遍历一次就 OK 了

举个栗子：

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220917205724.png)

- 如上图，比较的时候会一层一层比较，也就是图中蓝框的比较
- 到第二层的时候我们发现，`L` 带着`B`和`C`从`A`的下面，跑到了`R`的下面，按理说应该把`L`移到`R`的下方，但这样会牵扯到跨层级比较，有可能在层级上移动的非常多，导致时间复杂度陡然上升
- 所以在这里，React 会删掉整个 `A`，然后重新创建，但这种情况在实际中会非常少见

注意：**保持 DOM 的稳定**会有助于性能的提升，合理的利用显示和隐藏效果会更好，而不是真正的删除或增加 DOM 节点

### component diff

React 对于组件的策略有两种方式，分别是相同类型的组件和不同类型的组件

- 对同种类型组件对比，按照**层级比较**继续比较**虚拟 DOM 树**即可
- 对于不同组件来说，React 会直接判定该组件为**dirty component（脏组件）**，无论结构是否相似，**只要判断为脏组件就会直接替换整个组件的所有节点**

举个栗子：

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220917210246.png)

在比较时发现`D => G`，虽然两个组件的结构非常相似，React 判断这两个组件并不是同一个组件（dirty component），就会直接删除` D`，重新构建` G`,在实际中，两个组件不同，但结构又非常相似，这样的情况会很少的

### element diff

对于同一层级的一子自节点，通过唯一的 **key** 进行比较

当所有节点处以同一层级时，React 提供了三种节点操作：

- 1、插入（INSERT_MARKUP）
- 2、移动（MOVE_EXISTING）
- 3、删除（REMOVE_NODE）

下面通过具体的示例了解 diff 的过程

#### **情形一：新旧集合中存在相同节点但位置不同时，如何移动节点**

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220917220510.png)

新老集合所包含的节点，如下图所示，新老集合进行 `diff` 差异化对比，通过 `key` 发现新老集合中的节点都是相同的节点，因此无需进行节点删除和创建，只需要将老集合中节点的位置进行移动，更新为新集合中节点的位置，此时 React 给出的 `diff` 结果为：`B、D` 不做任何操作，`A、C` 进行移动操作，即可。

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220917220635.png)

具体的流程我们用一张表格来展现一下：

| newIndex | 节点 | index | lastIndex | 操作                                                 |
| -------- | ---- | ----- | --------- | ---------------------------------------------------- |
| 0        | B    | 1     | 0         | index(1) > lastIndex(0),lastIndex=index              |
| 1        | A    | 0     | 1         | index(0) < lastIndex(1),节点 A 移动至 index(1)的位置 |
| 2        | D    | 3     | 1         | index(3) > lastIndex(1),lastIndex=index              |
| 3        | C    | 2     | 3         | index(2) < lastIndex(3),节点 C 移动至 index(2)的位置 |

- `newIndex`: 新集合的遍历下标。
- `index`：当前节点在老集合中的下标。
- `lastIndex`：在新集合访问过的节点中，其在老集合的最大下标值。

> 注意：`lastIndex` 一开始默认值是`0`，它会与旧元素位置`index`进行比较，比较完后，会改变自己的值的（取 `index` 和 `lastIndex` 的较大数）。

**操作一栏中只比较 index 和 lastIndex：**

- 当 index > lastIndex 时，将 index 的值赋值给 lastIndex
- 当 index = lastIndex 时，不操作
- 当 index < lastIndex 时，将当前节点移动到 newIndex 的位置

**描述 diff 的差异对比过程如下：**

- （1）看着上图的 B，React 先从新中取得 B，然后判断旧中是否存在相同节点 B，当发现存在节点 B 后，就去判断是否移动 B。 B 在旧 中的 index=1，它的 lastIndex=0，不满足 index < lastIndex 的条件，因此 B 不做移动操作。此时，一个操作是，lastIndex=(index,lastIndex)中的较大数=1.

- （2）看着 A，A 在旧的 index=0，此时的 lastIndex=1（因为先前与新的 B 比较过了），满足 index<lastIndex，因此，对 A 进行移动操作，此时 lastIndex=max(index,lastIndex)=1。

- （3）看着 D，同（1），不移动，由于 D 在旧的 index=3，比较时，lastIndex=1，所以改变 lastIndex=max(index,lastIndex)=3

- （4）看着 C，同（2），移动，C 在旧的 index=2，满足 index<lastIndex（lastIndex=3），所以移动。

由于 C 已经是最后一个节点，所以 diff 操作结束。

### **情形二：新集合中有新加入的节点，旧集合中有删除的节点**

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220919200446.png)

具体的流程我们用一张表格来展现一下：

| newIndex | 节点 | index | lastIndex | 操作                                                    |
| -------- | ---- | ----- | --------- | ------------------------------------------------------- |
| 0        | B    | 1     | 0         | index(1) > lastIndex(0)，lastIndex=index(1)             |
| 1        | E    | -     | 1         | index 不存在，添加节点 E 至 newIndex(1)的位置           |
| 2        | C    | 2     | 1         | 不操作                                                  |
| 3        | A    | 0     | 3         | index(0) < lastIndex(3),节点 A 移动至 newIndex(3)的位置 |

> 注：最后还需要对旧集合进行循环遍历，找出新集合中没有的节点，此时发现存在这样的节点 D，因此删除节点 D，到此 diff 操作全部完成。

同样操作一栏中只比较 index 和 lastIndex，但是 index 可能有不存在的情况：

- index 存在

  - 当 index > lastIndex 时，将 index 的值赋值给 lastIndex
  - 当 index = lastIndex 时，不操作
  - 当 index < lastIndex 时，将当前节点移动到 newIndex 的位置

- index 不存在
  - 新增当前节点至 newIndex 的位置

描述 diff 的差异对比过程如下：

- （1）B 不移动，index=1，它的 lastIndex=0，不满足 index < lastIndex 的条件，因此 B 不做移动操作，更新 lastIndex=(index,lastIndex)中的较大数=1
- （2）新集合取得 E，发现旧不存在，故在 lastIndex=1 的位置 创建 E，更新 lastIndex=1
- （3）新集合取得 C，C 不移动，更新 lastIndex=2
- （4）新集合取得 A，A 移动，同上，更新 lastIndex=2
- （5）新集合对比后，再对旧集合遍历。判断 新集合 没有，但 旧集合 有的元素（如 D，新集合没有，旧集合有），发现 D，删除 D，diff 操作结束。

### **diff 的不足与待优化的地方**

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220919201704.png)

若新集合的节点更新为：D、A、B、C，与老集合对比只有 D 节点移动，而 A、B、C 仍然保持原有的顺序，理论上 diff 应该只需对 D 执行移动操作，然而由于 D 在老集合的位置是最大的，D 不移动，但它的 index 是最大的，导致更新 lastIndex=3，从而使得其他元素 A,B,C 的 index<lastIndex，导致 A,B,C 都要去移动。

> 建议：在开发过程中，尽量减少类似将最后一个节点移动到列表首部的操作，当节点数量过大或更新操作过于频繁时，在一定程度上会影响 React 的渲染性能。
