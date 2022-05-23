---
title: DvaJS入门
date: 2021-02-24 19:38:29
tags:
  - React
  - 前端
categories:
  - React
  - 前端
permalink: /pages/e6d6cc/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

## 简介

dva 首先是一个基于 redux 和 redux-saga 的数据流方案，然后为了简化开发体验，dva 还额外内置了 react-router 和 fetch ，所以也可以理解为一个轻量级的应用框架

来看个基本的代码结构

```js
const UserModel = {

  namespace: 'users',
  // 指定初始数据
  state: {},
  // 以 key/value 格式定义 reducer。用于处理同步操作，**唯一**可以修改 state 的地方。由 action 触发。是数据返回页面的唯一出口
  reducers: {
    getList(state, action) {
      // return newState  返回页面需要的数据
    }
  }
  // 处理异步函数，在effect中可以使用put来调用reducer中的函数
  // effect中没有返回值，需要通过put传递给同级的effects或reducers，不能直接返回数据
  effects: {
    *function_name(action, effects) {
      // yield put(state)
    }
  }
  // 再出发特定状态的时候调用，可以是当前的时间、服务器的websocket连接、keyboard输入. geolocation变化、 history路由变化等等。
  subscriptions: {
    // done可以省略
    function_name({ dispatch, history }, done) {
      // body...
    }
  }
}
export default UserModel
```

## action

在以上的代码`reducers`中，参数 state 是数据，action 是一个对象

action 中包含 `type` 和 `payload`

```js
// action:
{
  type:'getList',
  // payload没有可以不写
  payload:{}
}
// 因此，reducers代表的是 ：
getList(state, {type,payload})
```

## effects 中的参数细节

### put

在 `effects` 中，我们使用 `put` 来执行下一步动作，put 的对象可以是 **effects** 或 **reducers**。put 传递的是一个 `state`

```js
*function_name(action, effects) {
  yield put(state)
  //put的参数可解构为 ↓
  yield put({
    type:'getList',
    payload:data
  })
}
```

经过上面代码的操作后，put 中传递的对象就给到了 `reducers` 的 `action` 中。

### effects

在上面的代码中，我们发现传递的**第二个参数**是`effects`，这个参数好像没有用到，到底是什么东西？

其实，**effects 里面包含着 put 和 call**，方法也可以写成:

```js
*function_name(action, {put,call }) {

}
```

### put 和 call 区别

- call：执行**异步函数**(在下面 effects 调用接口的小节中会讲到)
- put：发出一个 Action，类似于 dispatch,用来调用**effects** 或 **reducers**

## 触发 effects

现在，我们对整个 dva 的数据传递过程有了一定了解，但是，我们要怎么触发 effect 来完成整个过程呢？ 我们要使用到`dispatch`

```js
// 在页面或者subscriptions中
dispatch({
  type: 'getList',
});
```

## effects 中调用接口(call、request 的使用)

### request

`request`是一个内置的请求方法,类似 axios

关于`request`的使用，可以看 github 的官网 [umi-request](https://github.com/umijs/umi-request/blob/master/README_zh-CN.md)

### 接口交互

effects 作为一个处理异步逻辑的地方，肯定会把接口交互放在这里，所以我们新建一个`service.js`文件来进行 ajax 请求

```js
//service.js
// 无参
export const getRemoteList = async (params) => {
  // 函数体需要整个return回去，因为call接收的是函数的返回值，如果只在resolve里面做return，call接收不到
  return request('xxx/xxx', {
    method: 'get',
  })
    .then((res) => {
      return res;
    })
    .catch((err) => {
      console.log(err);
    });
};
//带参
export const editRecard = async ({ value1, value2 }) => {
  return request(`xxx/xxx${value1}`, {
    method: 'put',
    data: value2,
  })
    .then((res) => {
      return res;
    })
    .catch((err) => {
      console.log(err);
    });
};
```

然后，在 model.js 中进行操作

```js
//model.js

//导入
import  getRemoteList from './service';
//使用
*function_name(action, {put,call}) {
  //--------关键代码----------------
  //无参
  const data = yield call(getRemoteList)
  //带参
  const data2 = yield call(editRecard,{'参数1','参数2'});
  //--------------------------------
  yield put({
    type:'getList',
    payload:data
  })
}
//如果发现传递的数据有问题，可以在put传递的时候包一层{}.使用的时候{user.data}
  yield put({
    type:'getList',
    payload:{data}
  })
```

## 与组件的 connect

在组件中使用时，我们需要建立连接

```js
//这个users是model只中namespace为user的model
const mapstateToProps = ({ users }) => ({
  users,
});
export default connect(mapstateToProps)(index);
```

当然，我们可以暴力一点，使用匿名函数直接整合在一起

```js
export default connect(({ users }) => ({
  users,
}))(index);
// 换成一行，比较难看，不够清晰
export default connect(({ users }) => ({ users }))(index);
```

然后就可以把 `users` 拿来使用了,同时还有`dispatch`

```js
const index = ({ users,dispatch }) => {
  .....
  dispatch({
    //**必须加上namespace**
    type:'users/xxx',
    payload:data
  })
  .....
}
```
