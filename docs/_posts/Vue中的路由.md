---
title: Vue中的路由
date: 2020-07-28 16:16:41
tags:
  - Vue
categories:
  - 前端
  - Vue
copyright: true
permalink: /pages/2fb933/
sidebar: auto
author:
  name: 杨雨翔
  link: https://gitee.com/xiang0515
---

## 路由

### 后端路由

- 概念:根据不同的用户 URL 请求，返回不同的内容
- 本质:URL 请求地址与服务器资源之间的对应关系

### 前端路由

- 概念:根据不同的用户事件，显示不同的页面内容
- 本质:用户事件与事件处理函数之间的对应关系

## Vue Router

&emsp;&emsp;vue Router(官网: https://router.vuejs.org/zh/ )是 Vue.js 官方的路由管理器。

### 基本使用步骤

- 1.引入相关的库文件
- 2.添加路由链接
- 3.添加路由填充位
- 4.定义路由组件
- 5.配置路由规则并创建路由实例
- 6.把路由挂载到 Vue 根实例中

我们来看最简单的例子

```html
<!-- 被vm实例所控制的区域-->
<div id="app">
  <router-link to=" /user">User</router-link>
  <router-link to="/register">Register</router-link>
  <!--路由占位符-->
  <router-view></router-view>
</div>
```

```js
//定义路由组件
const User = {
    template: '<h1>User 组件</h1>'
}
const Register = {
    template: '<h1>Register 组件</h1>'
}

//创建路由实例对象
var router = new vueRouter ({
    // routes是路由规则数组
    routes: [
    //每个路由规则都是一个配置对象，其中至少包含path和component两个属性:
    //path表示当前路由规则匹配的hash地址
    // component表示当前路由规则对应要展示的组件
    {path:'/user', component: User},
    {path:'/register',component: Register}
})
//把路由挂载到Vue根实例中
//创建vm实例对象
const vm = new Vue({
    //指定控制的区域
    el: '#app'，
    data:{}
    //挂载路由实例对象
    // router: router  可简写
    router
)}
```

### 路由重定向

路由重定向指的是 ∶ 用户在访问地址 A 的时候，强制用户跳转到地址 c，从而展示特定的组件页面;
通过路由规则的`redirect`属性，指定一个新的路由地址，可以很方便地设置路由的重定向:

```js
var router = new vueRouter ({
    routes: [
        //其中，path表示需要被重定向的原地址，redirect表示将要被重定向到的新地址
        {path:'/', redirect: '/user'}，
        {path:'/user', component: user},
        {path:'/ register', component: Register}
    ]
})
```

### 嵌套路由

嵌套路由功能分析

- 点击父级路由链接显示模板内容
- 模板内容中又有子级路由链接
- 点击子级路由链接显示子级模板内容

如图
![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/嵌套路由.png)

#### 父路由组件模板

- 父级路由链接
- 父组件路由填充位

```html
<p>
    <router-link to="/user">User</router-link>
    <routir-link to="/register">Register</router-link>
</p>
<div>
    <!--控制组件的显示位置-->
    <router-view></router-view>
</div>
```

#### 子级路由模板

- 子级路由链接
- 子级路由填充位

```js
const Register ={
    template: `<div>
    <h1>Register组件</h1>
    <hr/>
    <router-link to=" /register/tab1" >Tab1</router-link>
    <router-link to=" /register/tab2" >Tab2</router-link>
    <!--子路由填充位置 -->
    <router-view />
    </ div>`
}
corst Tab1 = {
    template: '<h3>tab1子组件</h3>'
}
const Tab2 = {
    template: '<h3>tab2子组件</h3>'
}
```

#### 嵌套路由配置

```js
// ------------------第一种配置方法----------------------------
//父级路由通过children属性配置子级路由
const router = new vueRouter ([
    routes:[
    { path: '/user', component: User },
    { path: '/register',component: Register,
    children:[
        { path: '/register/tab1', component: Tab1 }，
        { path: '/register/tab2', component: Tab2 }
    ]}
}) //通过children属性，为/register添加子路由规则

//-------------------第二种配置方法-----------------------------
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User,
      children: [
        // 当 /user/:id/profile 匹配成功，
        // UserProfile 会被渲染在 User 的 <router-view> 中
        { path: 'profile', component: UserProfile},
        // 当 /user/:id/posts 匹配成功
        // UserPosts 会被渲染在 User 的 <router-view> 中
        { path: 'posts',component: UserPosts}
      ]
    }
  ]
})

```

**要注意，以 / 开头的嵌套路径会被当作根路径。 这让你充分的使用嵌套组件而无须设置嵌套的路径。**

### 动态路由匹配

应用场景:通过动态路由参数的模式进行路由匹配

```js
var router = new vueRouter({
    routes: [
    //动态路径参数以冒号开头
    path:{'/user/:id', component: user }
    ]
})
const User = {
    //路由组件中通过$route.params获取路由参数
    template: '<div>user的id为：{{ $route.params.id }}</div>'
}
```

### 路由传参

#### 最简单的路由传参方法

```html
<!-- 定义一个路由参数 -->
{ path: '/user/:id'}
<!-- 传值 -->
<router-link to="/user/123"></router-link>
<!-- 取值 -->
this.$route.params.id

<!-- query传值,指通过?后面的拼接参数传值 -->
<router-link to="/user?id=123"></router-link>
<!-- 取值 -->
this.$route.query.id
```

#### 利用 props

&emsp;&emsp;刚刚说了利用动态路由匹配可以进行路由的传参，但是$route 与对应路由形成高度耦合，不够灵活，所以可以使用`props`将组件和路由解耦

#### props 的值为布尔类型

```js
const router = new vueRouter({
  routes: [
    //如果props被设置为true，route.params将会被设置为组件属性
    { path: '/user/:id', component: User, props: true },
  ],
});
const User = {
  props: ['id'], //使用props接收路由参数,就不需要通过$route.params
  template: '<div>用户ID:{{ id }}</div>', //使用路由参数
};
```

#### 2.props 的值为对象类型

```js
const router = new vueRouter ( {
    routes: [
    //如果props是一个对象，它会被按原样设置为组件属性
    ( path: '/user/:id',component: User,props: { uname: 'lisi'，age: 12 }}//此时，无法访问id的值
})
const User = {
    props:['uname ', 'age' ],
    template: '<div>用户信息:{ uname + '---' + age}}</div>'
}
```

#### props 的值为函数类型

我们现在又想获取 id 的值，又想获取 name 和 age，怎么办呢

```js
constrouter =new vueRouter ({
    routes: [
        //如果props 是一个函数，则这个函数接收route对象为自己的形参
        {path:'/user/ :id',
        component: User,
        //route为动态参数对象。在路由url后面有几个参数项，route里就有几个参数值
        props: route =>([ uname: 'zs'，age: 20,id: route.params.id })}
    ]
})
const User = {
    props:[ 'uname ', 'age', 'id']，
    template: 'div>用户信息:[{ uname + '---' + age + '---' + id]]</div>'
}
```

### 命名路由

有时候，通过一个名称来标识一个路由显得更方便一些，特别是在链接一个路由，或者是执行一些跳转的时候。你可以在创建 `Router` 实例的时候，在 `routes` 配置中给某个路由设置名称。

```html
<p>
    <!-- <router-link to="/user">User</router-link> 传统路由 -->
    <routir-link :to="{name:'user',params:{id:3}}">Register</router-link>
    <!-- name:路由名称  params:路由需要携带的参数 -->
</p>
<div>
    <!--控制组件的显示位置-->
    <router-view></router-view>
</div>
```

```js
constrouter =new vueRouter ({
    routes: [
        //如果props 是一个函数，则这个函数接收route对象为自己的形参
        {
            name:'user'//**给路由起一个名字**
            path:'/user/ :id',
            component: User,
            //route为动态参数对象。在路由url后面有几个参数项，route里就有几个参数值
            props: route =>([ uname: 'zs'，age: 20,id: route.params.id })
        }
    ]
})
const User = {
    props:[ 'uname ', 'age', 'id']，
    template: '<div>用户信息:{{ uname + '---' + age + '---' + id}}</div>'
}
```

## 路由的编程式导航

### 页面导航的两种方式：

- 声明式导航:通过点击链接实现导航的方式，叫做声明式导航
  例如:普通网页中的`<a></a>`链接或 vue 中的`<router-link></router-link>`

- 编程式导航:通过调用 JavaScript 形式的 API 实现导航的方式，叫做编程式导航
  例如:普通网页中的 location.href
  心

### 编程式导航基本用法

常用的编程式导航 API 如下:

- this.$router.push('hash 地址')
- this.$router.replace('hash 地址')
- this.$router.go(n)

```js
//push的用法
const User = {
    template: `<div>
        <button @click="goRegister">跳转到注册页面</button>
    </div>`,
    methods:{
    goRegister: function () {
    //用编程的方式控制路由跳转
    this.$router.push('/register' );
}

//replace的用法
//跟 router.push 很像，唯一的不同就是，它不会向 history 添加新记录，而是跟它的方法名一样 —— 替换掉当前的 history 记录。

//go的用法
const Register = {
    template: `<div>
        <h1>Register组件</h1>
        <button @click="goBack">后退</button>
    </div>`,
    methods: {
        goBack(){
            this.$router.go(-1)
        }
    }
}
```

`$router.push({path:''})`本质是向 history 栈中添加一个路由，在我们看来是 切换路由，但本质是在添加一个`history`记录

### 编程式导航的参数规则

`router.push()`方法的参数规则

```js
//字符串(路径名称)
router.push('/home');
//对象
router.push({ path: '/home' });
//命名的路由(传递参数)
router.push({ name: '/user', params: { userId: 123 } });
//带查询参数，变成/register?uname=lisi
router.push({ path: '/register', query: { uname: 'lisi' } });
```

## hash 和 history

### hash

- hash 虽然出现在 URL 中，但不会被包括在 HTTP 请求中，对后端完全没有影- 响，因此改变 hash 不会重新加载页面。
- hash 模式下，仅 hash 符号之前的内容会被包含在请求中，如 http://www.- npc.com，因此对于后端来说，即使没有做到对路由的全覆盖，也不会返回 - 404 错误。
- hash 设置的新值必须与原来不一样才会触发动作将记录添加到栈中。
- hash 只可修改 ## 后面的部分，因此只能设置与当前 URL 同文档的 URL。
- hash 只可添加短字符串。

### history（服务器环境下才有效果）

- pushState() 设置的新 URL 可以是与当前 URL 同源的任意 URL；；
- pushState() 设置的新 URL 可以与当前 URL 一模一样，这样也会把记添加到栈中；
- pushState() 通过 stateObject 参数可以添加任意类型的数据到记中；；
- pushState() 可额外设置 title 属性供后续使用。

## 路由导航守卫

### 全局守卫

路由导航守卫可以用于在前端路由跳转之前判断用户的登录状态。写一个 vue_shop 中的例子：

```js
//在router.js实例中 挂载一个全局路由导航守卫
router.beforeEach((to，from,next)=>{
    // to 将要访问的路径
    // from 代表从哪个路径跳转而来
    // next是一个函数，表示放行
    //next()放行next( '/login'）强制跳转
    if (to.path === '/login') return next()
    //获取token
    const tokenStr = window.sessionStorage.getItem("token")
    //不存在token，返回登录页
    if (!tokenstr) return next('/login')
    //验证通过，放行
    next()
})
```

**确保 `next` 函数在任何给定的导航守卫中都被严格调用一次。它可以出现多于一次，但是只能在所有的逻辑路径都不重叠的情况下，否则钩子永远都不会被解析或报错。**这里有一个在用户未能验证身份时重定向到 /login 的示例：

```js
router.beforeEach((to, from, next) => {
  if (to.name !== 'Login' && !isAuthenticated) next({ name: 'Login' });
  else next();
});
```

### 路由独享守卫

可以在路由配置上直接定义 `beforeEnter` 守卫：

```js
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {
        // ...
      },
    },
  ],
});
```
