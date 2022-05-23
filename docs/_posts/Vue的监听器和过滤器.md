---
title: Vue的监听器和过滤器
date: 2020-08-12 17:09:54
tags:
  - Vue
categories:
  - 前端
  - Vue
copyright: true
permalink: /pages/dbd7cb/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

## 监听器

监听器可以监听数据的变化，当某个数据改变时，可以触发某个函数

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/监听器.png)

来看看监听器的简单用法

```js
//定义数据  在html中用v-model绑定
data{
    firstName:'xxx'
    lastName:'yyy'
    fullName:''
}
watch: {
    //监听firstname变化
    firstName: function (val) {
        // val表示变化之后的值
        this.fullName = val + this.lastName ;
    },
    //监听lastname变化
    lastName: function (val) {
        this.fullName = this.firstName + val;
    }
}
```

其实，对于这个例子，我们用计算属性也可以实现

```js
computed:{
    fullName: function(){
        return this.firstName +''+ this.lastName;
    }
}
```

反而计算属性更简单。对于这些**字符串操作，更适合计算属性做**，**监听器的使用场景主要在异步操作和弹窗提示**等功能中

&emsp;&emsp;现在我们来做一个小案例，需求:输入框中输入姓名，**失去焦点**时验证是否存在，如果已经存在，提示从新输入，如果不存在，提示可以使用。
![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/监听器案例.png)
我们需要做到以下东西：

- 通过 v-model 实现数据绑定
- 需要提供提示信息
- 需要侦听器监听输入信息的变化
- 需要修改触发的事件

我们先写个表单

```html
<div>
  <span>用户名:</span>
  <span>
    <input type="text" v-model.lazy="uname" />
  </span>
  <span>{{tip}}</span>
</div>
```

然后来写监听器，注意以下几点

- 采用侦听器监听用户名的变化
- 调用后台接口进行验证
- 根据验证的结果调整提示信息

```js
var vm = new Vue({
    el:'#app',
    data: {
        uname:'',
        tip:''
    },
    methods:{
    checkName: function(uname){
        //调用接口，但是可以使用定时任务的方式模拟接口调用
        var that = this;//可以用箭头函数
        setTimeout(function(){
            //模拟接调用
            if(uname == 'admin'){
                that.tip ='用户名己经存在，请更换一个';
            }else{
                that.tip ='用户名可以使用'
            }
        },2000);
    },
    watch:{
        uname: function(val){
            //调用后台接口验证用户名的合法性
            this.checkName(val)
            this.tip = '正在验证...'
        }
    }
});
```

## 过滤器

### 过滤器的作用是什么?

格式化数据，比如将字符串格式化为首字母大写，将日期格式化为指定的格式等

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/过滤器.png)

### 自定义过滤器

```js
vue.filter(upper, function (value) {
  //把过滤器加到哪里，value就是哪里的数据
  //过滤器业务逻辑  首字母大写
  return val.charAt().toUpperCase() + val.slice(1);
});
vue.filter(lower, function (value) {
  //过滤器业务逻辑  首字母小写
  return val.charAt().toLowerCase() + val.slice(1);
});
```

### 过滤器的使用

```html
<div>{{msg | upper}}</div>
<div>{{msg | upper | lower}}</div>
<div v-bind:id="id | formatId"></div>
```

### 局部过滤器

```js
//效果和全局过滤器一样，但是在组件内部才能使用
filters: {
    upper: function(val){
        return val.charAt(0).toUpperCase()+val.slice(1);
    }
}
```

### 带参数的过滤器

我们想把当前时间(如 2020-8-12T09:20:15)的时间改成 2020-8-12 可以这么做：

```js
//第一个参数是要过滤的参数，第二个是过滤器携带的参数
Vue.filter('format', function(value,arg) {
    //如果是这个格式
    if(arg =='yyyy-MM-dd'){
        var ret ='';
        ret+= value.getFullYear()+'-'+(value.getMonth()+ 1)+'-'  +value.getDate();
        return ret
    }、
    //如果不是这个格式
    return value;
}

```

过滤器的使用

```html
<div>{{date | format('yyyy-MM-Nd')}}</div>
```
