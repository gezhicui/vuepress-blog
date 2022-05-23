---
title: Vuex
date: 2020-08-15 15:05:46
tags:
  - Vux
categories:
  - Vuex
copyright: true
permalink: /pages/067c99/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

## Vuex

官方解释:`Vuex是一个专为Vuejs应用程序开发的状态管理模式。`

它采用**集中式存储管理**应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。

**状态管理到底是什么?**

- 状态管理模式、集中式存储管理这些名词听起来就非常高大上，让人捉摸不透。
- 其实，你可以简单的将其看成把需要多个组件共享的变量全部存储在一个对象里面。
- 然后，将这个对象放在顶层的 Vue 实例中，让其他组件可以使用。
- 那么，多个组件就可以共享这个对象中的所有变量属性了

**等等，如果是这样的话，为什么官方还要专门出一个插件 Vuex 呢?难道我们不能自己封装一个对象来管理吗?**
比如说我们定义一个`shareObj`对象，挂载到 Vue 的原型上

```js
const shareObj = {
  name: 'yang',
};
Vue.prototype.shareObj = shareObj;
```

当然可以，只是我们要先想想 VueJS 带给我们最大的便利是什么呢?没错，就是响应式。如果这么做了，我们更新`shareObj`中的值，**没办法触发**页面中的值刷新。因为他没被加到 Vue 的响应式系统里，而 data 中的数据被加载到了响应式系统里，所以能触发刷新。

### 管理什么状态

有什么状态时需要我们在多个组件间共享的呢?
&emsp;&emsp;如果你做过大型开发，你一定遇到过多个状态，在多个界面间的共享问题。比如**用户的登录状态、用户名称、头像、地理位置信息**等等。比如**商品的收藏、购物车中的物品**等等。
&emsp;&emsp;这些状态信息，我们都可以放在统一的地方，对它进行保存和管理，而且它们还是响应式的

## 安装

> npm vuex --save

## 使用

安装完成后，我们在`src`目录下新建`store`文件夹，下面放个`index.js`存放 Vuex

```js
// store/index.js
import vue from 'vue';
import Vuex from 'vuex';
// 1.安装插件
vue.use(vuex);
// 2.创建对象
const store = new Vuex.Store({
  //存放状态
  state: {
    counter: 0,
  },
  //修改状态的方法
  mutations: {},
  //异步方法
  actions: {},
  //计算属性
  getters: {},
  //模块
  modules: {},
});
// 3.导出store独享
export default store;

//main.js引入
import store from './store';
//在vue实例中加入store  这样就可以在全局使用$store使用vuex
new Vue({
  el: '#app',
  store,
  render: (h) => h(App),
});
```

创建完后，我们就可以在所有组件中使用共享的状态了

```html
<template>
  <div>{{$store.state.counter}}</div>
</template>
```

这时候，我们想修改`counter`中的值，有啥办法，可能会想到这种办法：

```html
<button @click="$store.state.counter++"></button>
```

这种方法也可以改变`counter`的值，但是**不推荐！** 来看一张官方的图

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/vuex.png)

**我们应该通过 Mutation 来修改 state**

&emsp;&emsp;Vue 官方为我们提供了一个`Devtools`插件，在多个页面对 Vuex 进行修改时，我们能知道到底是哪个界面修改了 vuex 的东西，这样维护起来的时候比较方便
&emsp;&emsp;如果我们像上面的例子一样绕过了`Mutations`，直接对`state`进行修改，我们就是不知道到底哪个界面修改了`state`

当然，我们在修改的时候也可以跳过`Action`
![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/vuex跳过action.png)

&emsp;&emsp;`Action`的作用是当你在修改`Mutation`时，如果有异步操作(发送网络请求)，放在`Action`中进行。`Devtools`无法跟踪异步操作，所以把异步放在`Action`中执行完了发给`Mutation`，让`Devtools`可以跟踪，让调试更加方便

## Mutatuons 使用

Vuex 的 store 状态的**更新唯一方式:提交 mutation**

下面我们来通过 mutation 修改 state,首先在 mutation 中添加方法

```js
//在store/index.js中
const store = new Vuex.Store({
  state: {
    counter: 0,
  },
  mutations: {
    //state为默认参数
    increament(state) {
      state.counter++;
    },
    decreament(state) {
      state.counter--;
    },
    increamentCount(state, count) {
      state.counter += count;
    },
  },
  actions: {},
  getters: {},
  modules: {},
});
```

然后在使用到 vuex 的组件中，先定义事件

```html
<button @click="addition">+</button>
<button @click="subtraction">-</button>
<!-- 带参传递 -->
<button @click="addCount(5)">+5</button>
<button @click="addCount(10)">+10</button>
```

定义事件处理函数

```js
methods:{
    addition(){
        //commit提交对应mutations
        this.$store.commit('increment')
    },
    decreament(){
        //提交对应mutations
        this.$store.commit('decreament')
    },
    //带参传递，参数可以是对象 这里的参数有个专业名称：mutation的payload(负荷，负载)
    addCount(count){
        //提交对应mutations
        this.$store.commit('increamentCount',count)
    }
}
```

## Action 的使用

&emsp;&emsp;通常情况下,Vuex 要求我们`mutation`中的方法必须是同步方法。

&emsp;&emsp;主要的原因是当我们使用`devtools`时,`devtools`可以帮助我们捕捉`mutation`的快照。但是如果是异步操作,那么`devtools`将不能很好的追踪这个操作什么时候会被完成。
&emsp;&emsp;但是某些情况,我们确实希望在 Vuex 中进行一些异步操作,比如网络请求,必然是异步的.这个时候怎么处理呢?这时候，我们就要用到`action`
&emsp;&emsp;`action`类似于`mutation`,但是是用来代替`mutation`进行异步操作的。

Action 的基本使用如下：

```js
//在store/index.js中
const store = new Vuex.Store({
    state:{
        info:{
            name:'yang'
        }
    },
    mutations:{
        UpdateInfo(state){
            state.info.name = 'xiang'
        }
    }
    actions:{
        //mutations和getter默认参数为state,而actions为context
        //context为上下文
        actionUpdateName(context,payload){//payload只有带参传递过来的函数才需要写
            setTimeout(()=>{
                context.commit(UpdateInfo)
                console.log(payload)
            },1000)
        }
    },
    getters:{},
    modules:{}
})
```

然后在使用到 vuex 的组件中，先定义事件

```html
<button @click="UpdateName"></button>
<!-- 带参传递 -->
<button @click="addCount()">哈哈</button>
```

定义事件处理函数

```js
methods:{
    //  1  不带参数传递
    UpdateName(){
        //dispatch提交对应actions
        this.$store.dispatch('actionUpdateName')
    },
    //  2  带参传递，参数可以是对象payload(负荷，负载)
    UpdateName(){
        //提交对应mutations
        this.$store.dispatch('increamentCount','hahah')
    }
}
```

## Getters 使用

&emsp;&emsp;在使用`state`数据的时候，如果很经常要用到`state`数据的变形，如我现在经常要用到`counter`的平方，正常情况下我可以这么写

```html
<h2>{{$store.state.counter * $store.state.counter}}</h2>
```

但是这样太麻烦，我们可以定义`getters`。另一种方法是定义一个 computed 计算属性，这里不讲

```js
//store/index.js中
const store = new Vuex.Store({
  state: {
    counter: 110,
  },
  getters: {
    //state为默认参数
    powerCounter(state) {
      return state.counter * state.counter;
    },
  },
});
```

现在，我们就可以这样用 getters:

```html
<h2>{{$store.getters.powerCounte}}</h2>
```

getters 是默认不能传递参数的，如果想传递参数，可以 return 一个 function

```js
//<h2>{{$store.getters.Count(2)}}

getters:{
    Count(state){
        return function(num){
            return state.counter*num
        }
    }
}
```

## Modules 的使用

&emsp;&emsp;Module 是模块的意思，为什么在 Vuex 中我们要使用模块呢?

&emsp;&emsp;Vue 使用单一状态树,那么也意味着很多状态都会交给 Vuex 来管理。当应用变得非常复杂时,store 对象就有可能变得相当臃肿
&emsp;&emsp;为了解决这个问题,Vuex 允许我们将 store 分割成模块(Module)，而每个模块拥有自己的 state、mutation、action、getters 等

我们按以下形式组织代码：

```js
const moduleA = {
    state: { ... },
    mutations: { ...},
    actions: { ... },
    getters: { ...}
}
const moduleB = {
    state: { ... },
    mutations: { ...},
    actions: { ...}
}
const store = new Vuex.Store({
    modules: {
        a: moduleA,
        b: moduleB
    }
})
store.state.a // ->moduleA的状态
store.state.b // -> moduleB 的状态
```

## Vuex-store 文件夹的目录结构组织

全部内容都写在 index.js 里面太乱了，我们可以分开几个文件，比如我们把 mutations 抽离出去，在 store 文件夹下新建一个 mutations.js

```js
//mutations.js
export default {
    //mutations中的方法
}

//然后在index.js中引入
///index.js
import mutations from './mutations.js'

const store = new Vuex.Store({
    state,
    mutations,
    ...
})
```

## 数据的响应式原理

&emsp;&emsp;Vuex 的 store 中的 state 是响应式的，当 state 中的数据发生改变时, Vue 组件会自动更新

这就要求我们必须遵守一些 Vuex 对应的规则:

- 提前在 store 中初始化好所需的属性.
- 当给 state 中的对象添加新属性时，使用下面的方式:
  》方式一:使用 Vue.set(obj, 'newProp',123)
  》方式二:用心对象给旧对象重新赋值

&emsp;&emsp;我们在数据中预先定义好的的每一个属性都会被加入到响应式系统中，并对应着一个 Dep 这样的对象，这是一个观察者模式，可以监听数据有没有变化，一旦数据变化，他会去寻找哪些地方需要用到这个属性去刷新界面，然后通知这些地方做出改变

&emsp;&emsp;我们可以通过**Vue.set**方法让对象里加入数据且成为响应式的

```js
//Vue.set(修改的对象/数组  ，  对象的属性/数组的索引  ，  值)
//对象
Vue.set(state.info, 'address', 'Fuzhou');
//数组
Vue.set(sate.Arry, 0, 'Fuzhou');

//在组件中也可使用
this.$set(this.info, 'address', 'fuzhou');
```

&emsp;&emsp;this.$set和Vue.set的区别：
Vue.set 可以设置实例创建之后添加的新的属性，（在data里未声明的属性），而。this.$set 只能设置实例创建后存在的对象的属性。
