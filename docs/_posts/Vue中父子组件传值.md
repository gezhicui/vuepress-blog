---
title: Vue中父子组件传值和访问
date: 2020-07-26 12:25:49
tags:
  - Vue
categories:
  - 框架

permalink: /pages/69ae2b/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

在开发中，往往一些数据确实需要从上层传递到下层 ∶

&emsp;&emsp;比如在一个页面中，我们从服务器请求到了很多的数据。

&emsp;&emsp;其中一部分数据，并非是我们整个页面的大组件来展示的，而是需要下面的子组件进行展示。

&emsp;&emsp;这个时候，并不会让子组件再次发送一个网络请求，而是直接让**大组件(父组件)**将数据传递给**小组件(子组件)**。

如何进行父子组件间的通信呢?Vue 官方提到

- 通过`props`向子组件传递数据
- 通过事件向父组件发送消息

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/父子组件传值.png)

## 父传子 props

&emsp;&emsp;在组件中，使用选项 props 来声明需要从父级接收到的数据。props 的值有两种方式:

- 方式一:字符串数组，数组中的字符串就是传递时的名称。
- 方式二 ∶ 对象，对象可以设置传递时的类型，也可以设置默认值等。

```html
<!-- 父组件模板 -->
<div id="app">
  <!-- 传递动态props -->
  <cpn :cmovies="movies" :message="message"></cpn>
  <!-- 传递静态props -->
  <cpn cmovies="鳄鱼洗澡" message="hshshsh"></cpn>
  <!-- 传入数字、布尔值、数组、对象时，都需要用v-bind -->
</div>
<!-- 子组件模板 -->
<template id="cpn">
  <div>
    {{cmovies}} {{message}}
    <ul>
      <li v-for="item in cmovies">{{item}</li>
    </ul>
  </div>
</template>
```

```js
//子组件
const cpn = {
    template:'#cpn',
    //数组写法
    props: [ 'cmovies','message'],
    //对象写法（推荐）
    props:{
        cmovies:Array,
        //还可以设置默认值，如果没传参，则显示默认值
        messgae:{
            type:String,
            default:'哈哈哈'
        }
    },
    data(){
        //return
    }，
    methods:{
    }
}

//初始化vue实例 父组件
const app = new Vue({
    el:'#app',
    data: {
        message:'你好啊'，
        movies:['海王'，'海贼王'，'海尔兄弟']
    }，
    components: {
        cpn//cpn:cpn的简写
    }
})
```

此时，父组件中的 movies 和 message 已经传递到子组件中并可以展示了

**这里我们要注意`v-bind`的使用，在传值是我们什么时候加上`v-bind `，什么时候不加？**

### 传入 String 类型

```html
<!-- 传入的值title为一个常量(静态prop)时，不加v-bind(或者：) -->
<blog-post title="My journey with Vue"></blog-post>
<!-- 传入的值title为一个变量(动态prop)时，加v-bind(或者：) -->
<blog-post v-bind:title="titleValue"></blog-post>
```

### 传入 Number 类型

```html
<!-- 无论静态的'42'还是变量totalNumber(动态)的值为42，我们都需要 `v-bind` 来告诉 Vue -->
<blog-post v-bind:total="42"></blog-post>
<blog-post v-bind:total="totalNumber"></blog-post>
```

### 传入 Boolean 类型

```html
<!-- 无论静态的'false'还是变量booleanValue(动态)的值为false，我们都需要 `v-bind` 来告诉 Vue -->
<base-input v-bind:favorited="false">
  <base-input v-bind:favorited="booleanValue"></base-input
></base-input>
```

### 传入一个数组

```html
<!-- 无论静态的'[234, 266, 273]'还是变量commmetArray(动态)的值为[234, 266, 273]，我们都需要 `v-bind` 来告诉 Vue -->
<blog-post v-bind:comment-ids="[234, 266, 273]"></blog-post>
<blog-post v-bind:comment-ids="commmetArray"></blog-post>
```

### 传入一个对象

```html
<!-- 无论静态的"{name:'bob'}"还是变量postObject(动态)的值为{name:'bob'}，我们都需要 `v-bind` 来告诉 Vue -->
<blog-post v-bind:post="{name:'bob'}"></blog-post>
<blog-post v-bind:post="postObject"></blog-post>
```

## 子传父 $emit

```html
<!-- 父组件模板 -->
<div id="app">
  <!-- 监听子组件发射事件 -->
  <cpn @itemClick="cpnClick"></cpn>
</div>
<!-- 子组件模板 -->
<template id="cpn">
  <div>
    <!-- 点击触发切换子组件事件 -->
    <button v-for="item in categories" @click="btnClick(item)">
      {{item.name}
    </button>
  </div>
</template>
```

```js
//子组件
const cpn = {
    template:'#cpn',
    },
    data(){
        return{
            categories:[
                {id:'aaa',name='热门推荐'},
                {id:'bbb',name='手机数码'},
                {id:'ccc',name='家用家电'},
                {id:'ddd',name='电脑办公'}
            ]
        }
    },
    methods:{
        btnClick(item){
            //给父组件传值，告诉父组件这个按钮被点击了
            //发射事件
            this.$emit('itemClick')
            //同时，也可以携带参数
            this.$emit('itemClick',item)
        }
    }
}

//初始化vue实例 父组件
const app = new Vue({
    el:'#app',
    data: {
    }，
    components: {
        cpn//cpn:cpn的简写
    }
    methods:{
        //若无参数,item不写
        cpnClick(item){
            console.log("接收到子组件的点击事件")
        }
    }
})
```

## 单向数据流

原文链接：[vue 官方文档：单向数据流](https://cn.vuejs.org/v2/guide/components-props.html)

&emsp;&emsp;所有的 prop 都使得其父子 prop 之间形成了一个**单向下行绑定**：父级 prop 的更新会向下流动到子组件中，但是反过来则不行。这样会防止从子组件意外变更父级组件的状态，从而导致你的应用的数据流向难以理解。

&emsp;&emsp;额外的，每次父级组件发生变更时，子组件中所有的 prop 都将会刷新为最新的值。这意味着你不应该在一个子组件内部改变 prop。如果你这样做了，Vue 会在浏览器的控制台中发出警告。

&emsp;&emsp;这里有两种常见的试图变更一个 prop 的情形：

- **这个 prop 用来传递一个初始值；这个子组件接下来希望将其作为一个本地的 prop 数据来使用**。在这种情况下，最好**定义一个本地的 data property**并将这个 prop 用作其初始值：

```js
props: ['initialCounter'],
data: function () {
  return {
    counter: this.initialCounter
  }
}
```

- **这个 prop 以一种原始的值传入且需要进行转换**。在这种情况下，最好**使用这个 prop 的值来定义一个计算属性**：

```js
props: ['size'],
computed: {
  normalizedSize: function () {
    return this.size.trim().toLowerCase()
  }
}
```

## vue 中父子组件访问

### 父子组件访问子组件方式：$children

&emsp;&emsp;有时候我们需要父组件直接访问子组件，子组件直接访问父组件，或者是子组件访问根组件。

- 父组件访问子组件:使用$children或$refs (reference 引用)
- 子组件访问父组件 ∶ 使用$parent

&emsp;&emsp;我们先来看下$children 的访问

- this.$children 是一个数组类型，它包含所有子组件对象。
- 我们这里通过一个遍历，取出所有子组件的 message 状态。

现在，我们想在父组件中调用子组件的方法，就可以用$children

```js
//我们先定义一个子组件
components:{
    cpn:{
        template:'#cpn',
        data(){
            return{
                name:'我是子组件的name'
            }
        },
        methods:{
            showMessage(){
                console.log('showMessage')
            }
        }
    }
}

//父组件如下：
const app = new Vue({
    el: '#app',
    data: {
        message:'你好啊'
    }
    methods:{
        btnclick(){
            //重要的在这里！！！ 我们访问$children,就是访问组件对象
            console.log(this.$children);
            //访问第一个组件对象的showmessage方法，输出'showmessage'
            this.$children[0].showMessage()
            //访问第一个子组件的name属性
            console.log(this.$children[0].name)
            //循环访问子组件的name
            (let c of this.$children){
                console.log(c.name)
            }
        }
    }
}
```

```html
在html中定义模板
<div id="app">
  <cpn></cpn>
  <cpn></cpn>
  <cpn></cpn>
  <button @click="btnclick">按钮</button>
</div>
<template id="cpn">
  <div>我是子组件</div>
</template>
```

### 父子组件访问子组件方式 $ref

&emsp;**但是，在普通的开发中，我们一般使用$ref而不是用$children,**因为会有可能我们在开发中插入了一个子组件，这样 this.$chilren[n]就有可能会出错,我们就可以用到$ref 指定访问的组件

```html
<!-- 对上一小节中的html模板改造 -->
<div id="app">
  <cpn></cpn>
  <cpn ref="aaa"></cpn>
  <cpn></cpn>
  <button @click="btnclick">按钮</button>
</div>
```

我们在 btnclick 这个方法中访问

```js
methids:{
    btnClick(){
            console.log(this.$refs);
            //访问第一个组件对象的showmessage方法，输出'showmessage'
            this.$refs.aaa.showMessage()
            //访问第一个子组件的name属性
            console.log(this.$ref.aaa.name)
    }
}
```

### 子组件访问父组件方式$parent

我们写一个父子组件的嵌套

```js
components: {
    cpn:{
        template: '#cpn',
        data(){
            return {
                name:'我是cpn组件的name'
            }
        }
        components: {
            ccpn:{
                template: '#ccpn',
                methods: {
                    btnclick(){
                    // 1.访问父组件$parent
                        console.log(this.$parent);
                        console.log(this.$parent.name);//输出'我是cpm组件的name'
                    }
                }
            }
        }
    }
}
```

但是在开发中并**不建议使用**，因为组件本来就是为了复用，如果这样写就必须要求父组件中有对应的方法，**降低了复用性**。

### 访问根组件 $root

```js
console.log(this.$root);
```
