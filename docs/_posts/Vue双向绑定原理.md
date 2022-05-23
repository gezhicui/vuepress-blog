---
title: Vue响应式原理
date: 2020-10-06 21:15:06
tags:
  - Vue
categories:
  - 前端
  - Vue
copyright: true
permalink: /pages/5a5d59/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

## 参考文章

[Vue 响应式原理学习总结](https://juejin.cn/post/6932659815424458760).
[学习 Vue 源码系列](https://gitee.com/ykang2020/vue_learn).

先放一张图，这是响应原理的实现思路

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/响应式原理.png)

- 在 new 出来的 Vue 对象中，我们定义`data`来存放我们的数据
- 在`Observer`对象中，我们通过`Object.defineProperty`来劫持所有的 data 属性。其实就是给每一个属性放进其对应的`Dep`对象。当数据发生改变时，调用`Dep`中的`notify`方法通知`watcher`执行`update()`方法做出改变
- 在`Dep`对象中，有一个`subs`数组，数组中存放着所有监听着这个数据的`data`对象(watcher)

- 在`Complite`对象中，我们通过`{{数据}}`来解析 el 模板中的指令。在每个使用的地方创建一个`watcher`对象,并把这个对象放到`Dep`中

## 前提概念

想要知道数据的变化过程,我们需要知道:

- 侦测数据的变化
- 收集视图依赖了哪些数据
- 数据变化时，自动“通知”需要更新的视图部分，并进行更新

对应专业俗语分别是：

- 数据劫持 / 数据代理
- 依赖收集
- 发布订阅模式

为什么我们修改的数据能被侦听到？

```js
let user = {
  username: 'yang',
  type: '帅',
};
```

当我们修改`“帅”`时，我们修改的数据能被侦听到。这是因为我们利用了`Object.defineProperty`定义属性，我们通过这个方法让一个对象的属性变成可侦测的

```js
let content = '厚脸皮';

Object.defineProperty(user, 'type', {
  enumerable: true, //定义此属性是否出现在for in循环,以及object.keys()的便利中
  configurable: true, //定义此属性可以配置和删除

  //取数据的描述符
  get() {
    console.log('现在在取type属性的数据');
    return content;
  },
  set(newValue) {
    console.log('现在在存type属性的数据,存入的值是：' + newValue);
    content = newValue;
  },
});

//取数据
let str = user.type;
console.log(str);
//存数据
user.type = '有钱';
console.log(user.type);

/* 
现在在取type属性的数据
厚脸皮
现在在存type属性的数据,存入的值是：有钱
现在在取type属性的数据
有钱
*/
```

## 侦测对象的所有属性

现在，我们让一个对象的属性变成可侦测的，那么怎么让**所有**对象的属性都变成可侦测的呢？

我们可以通过定义一个`Observer`类，使用**递归**的方式让对象的所有属性变成可侦测的

```js
//我们现在把user对象变成可侦测的，首先定义一个Observer类
class Observer{
    constructor(value){
        this.value = value;
        if(Array.isArray(value)){
            //value是数组的时候，用数组的侦听方式
        }else if(typeof value === Object){
            //当value是对象的时候调用walk实现循环侦测属性
            this.walk(value)
        }
    }
    walk(obj){
        //获取对象的所有属性列表
        const keys = Object.keys(obj)
        for(let i = 0;i<keys.length;i++){
            dedefineReactive(obj,key[i])
        }
    }
}

function defineReactive(obj,key,value){//对象，属性，改变的值(上个例子中的content)
    //当只传入2个值的时候，可以通过obj[key]直接获取到第三个参数value值
    if(arguments.length==2){
        value = obj[key]
    }
    //如果属性的值value也是对象，那么也需要将其变为可侦测
    if(typeof value==='object'){
        new Observer(val)
    }

    Object.defineProperty(obj, key,{
        enumerable:true,//定义此属性是否出现在for in循环,以 及object.keys()的便利中
        configurable:true,//定义此属性可以配置和删除

        //取数据的描述符
        get(){
            console.log(`现在在取${key}属性的数据`)
            return value
        }
        set(newValue){
            if(value===newValue){
                //值没变化，不做处理
                return
            }
            console.log(`现在在存${key}属性的数据,存入的值是：`+newValue)
            value = newValue
        }

    })
}

let user = {
    username:'yang',
    type:'帅'
    friend:{
        username:'隔壁老王'
    }
}

let user1 = new Observer(user)
```

## 订阅者 Dep

Dep 收集依赖需要为依赖找一个存储依赖的地方，为此我们创建了 Dep,它用来收集依赖、删除依赖和向依赖发送消息等。

于是我们先来实现一个订阅者 Dep 类，用于解耦属性的依赖收集和派发更新操作，说得具体点，它的主要作用是用来存放 Watcher 观察者对象。我们可以把**Watcher 理解成一个中介的角色，数据发生变化时通知它，然后它再通知其他地方。**

Dep 的简单实现:

```js
class Dep {
  constructor() {
    /* 用来存放Watcher对象的数组 */
    this.subs = [];
  }
  /* 在subs中添加一个Watcher对象 */
  addSub(watcher) {
    this.subs.push(watcher);
  }
  /* 通知所有Watcher对象更新视图 */
  notify() {
    this.subs.forEach((watcher) => {
      watcher.update();
    });
  }
}
```

以上代码主要做两件事情：

- 用 `addSub`方法可以在目前的 Dep 对象中增加一个` Watcher` 的订阅操作；
- 用 `notify` 方法通知目前 Dep 对象的 subs 中的所有 Watcher 对象触发更新操作。
  所以当需要依赖收集的时候调用 addSub，当需要派发更新的时候调用 notify。调用也很简单：

```js
let dp = new Dep();
//在get函数中执行addsub，这样就能把所有用到这个属性的地方收集起来
dp.addSub(() => {
  console.log('emit here');
});

//在set函数中触发watcher，完成监听数据改变重新渲染页面
dp.notify();
```

## 观察者 Watcher

当属性发生变化后，我们要通知用到数据的地方，而使用这个数据的地方有很多，而且类型还不一样，既有可能是模板，也有可能是用户写的一个 watch,这时需要抽象出一个能集中处理这些情况的类。然后，我们在依赖收集阶段只收集这个封装好的类的实例进来，通知也只通知它一个，再由它负责通知其他地方。

Watcher 的简单实现：

```js
//定义一个地方存放当前的watcher ，随便什么全局的地方，window.target也行
Dep.target=null

class Watcher{
    constructor(data, expression, cb){
   // 初始化watcher实例时订阅数据
    this.value = this.get()
    this.cb = cb
    get() {
      //把target指向自己这个watcher
      Dep.target = this
      /*
      **由于之前劫持了对象的属性访问，这里访问对象属性会触发get方法,
      然后触发dep.addsub,把当前watcher放到对象属性的subs数组中**！！！核心
       */
      const value = data[expression]
      return value
    }
    update() {
      // 获得新值
      this.value = this.obj[this.key]
      // 我们定义一个 cb 函数，这个函数用来模拟视图更新，调用它即代表更新视图
      this.cb(this.value)
    }
}

```

## 完整代码

我们把 dep 和 watcher 整合一下，放到 defineReactive 中：

```js
function defineReactive(obj,key,value){
  const that = this
  // **负责收集依赖，并发送通知**
  let dep=new Dep()
  //如果value是对象，继续设置它下面的成员为响应式数据
  this.walk(value)
  Object.defineProperty(obj, key,{
    enumerable:true,
    configurable:true,
    //取数据的描述符
    get(){
      //收集依赖，Dep. target就是观察者对象
    Dep.target && dep.addsub(Dep.target )
        return value
    }
    set(newValue){
      if(value===newValue) return
      this.walk(newValue)
      value = newValue
      //发送通知，因为在set 方法中改变数据
      dep.notify()
    }
  })
}

let w1 = new Watcher(obj, 'a', (val, oldVal) => {
  console.log(`obj.a 从 ${oldVal}(oldVal) 变成了 ${val}(newVal)`)
})
```
