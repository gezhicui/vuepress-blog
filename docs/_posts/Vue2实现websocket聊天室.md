---
title: Vue2实现websocket聊天室
date: 2021-03-22 19:11:34
tags:
  - Vue
  - websocket
categories:
  - websocket
permalink: /pages/a8f052/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

前几天，我们使用了原生 JS 来实现了 websocket 聊天室，今天我们使用 vue 再实现一遍

首先，我们先使用脚手架搭建项目，关于 vue 脚手架的内容参考之前的博客。搭建好项目框架后，我们来重新捋一下实现聊天室的基本原理：

## 聊天室原理

### 前端

#### Login

view 组成：用户名输入/进入聊天室的按钮

逻辑处理流程：input username(6) -> 存储到 localstorage -> enter the chatting room

涉及到事件：

- open
- close
- error
- message 接收广播来的数据

#### Home

view 组成：列表/消息输入框/发送按钮

逻辑处理流程：localstorage -> username / message / id /消息时间 → 后端 socket 服务

### 后端

接收 -> 消息数据 -> 广播给所有连接到 socket 服务的客户端

涉及到事件：

- open
- close
- error
- connection
- message 接收客户端发送的消息数据->广播

## 编码实现

知道了前后端该做什么之后，我们就来看看编码的实现：

### 前端

Login.vue

```vue
<template>
  <div class="about">
    <input type="text" placeholder="请输入用户名" v-model="username" />
    <button @click="handleEnterBtnClick">进入聊天室</button>
  </div>
</template>

<script>
export default {
  name: 'Login',
  data() {
    return {
      username: '',
    };
  },
  mounted() {
    const username = localStorage.getItem('username');

    if (username) {
      this.$router.push('/');
      return;
    }
  },
  methods: {
    handleEnterBtnClick() {
      const username = this.username.trim();

      if (username.length < 6) {
        alert('用户名不小于6位');
        return;
      }

      localStorage.setItem('username', username);
      this.$router.push('/');
    },
  },
};
</script>
```

Home.vue

```vue
<template>
  <div class="home">
    <ul>
      <li v-for="item of msgList" :key="item.id">
        <p>
          <span>{{ item.user }}</span>
          <span>{{ new Date(item.dateTime) }}</span>
        </p>
        <p>消息：{{ item.msg }}</p>
      </li>
    </ul>

    <input type="text" placeholder="请输入消息" v-model="msg" />
    <button @click="handleSendBtnClick">发送</button>
  </div>
</template>

<script>
// new websocket实例
const ws = new WebSocket('ws://localhost:8000');

export default {
  name: 'Home',
  data() {
    return {
      msg: '',
      username: '',
      msgList: [],
    };
  },
  mounted() {
    this.username = localStorage.getItem('username');

    if (!this.username) {
      this.$router.push('/login');
      return;
    }

    // 若不bind this，这里是ws调用方法，this指向ws
    ws.addEventListener('open', this.handleWsOpen.bind(this), false);
    ws.addEventListener('close', this.handleWsClose.bind(this), false);
    ws.addEventListener('error', this.handleWsError.bind(this), false);
    ws.addEventListener('message', this.handleWsMessage.bind(this), false);
  },
  methods: {
    handleSendBtnClick() {
      const msg = this.msg;

      if (!msg.trim().length) {
        return;
      }

      ws.send(
        JSON.stringify({
          id: new Date().getTime(),
          user: this.username,
          dateTime: new Date().getTime(),
          msg: this.msg,
        })
      );

      this.msg = '';
    },
    handleWsOpen(e) {
      console.log('FE: WebSocket open', e);
    },
    handleWsClose(e) {
      console.log('FE: WebSocket close', e);
    },
    handleWsError(e) {
      console.log('FE: WebSocket error', e);
    },
    handleWsMessage(e) {
      const msg = JSON.parse(e.data);
      this.msgList.push(msg);
    },
  },
};
</script>
```

### 后端

引入 ws 库

> yarn add ws

`index.js`

```js
const Ws = require('ws');

((Ws) => {
  const server = new Ws.Server({ port: 8000 });

  const init = () => {
    bindEvent();
  };

  function bindEvent() {
    server.on('open', handleOpen);
    server.on('close', handleClose);
    server.on('error', handleError);
    server.on('connection', handleConnection);
  }

  function handleOpen() {
    console.log('BE: WebSocket open');
  }

  function handleClose() {
    console.log('BE: WebSocket close');
  }

  function handleError() {
    console.log('BE: WebSocket error');
  }

  function handleConnection(ws) {
    console.log('BE: WebSocket connection');

    ws.on('message', handleMessage);
  }

  function handleMessage(msg) {
    server.clients.forEach((c) => {
      c.send(msg);
    });
  }

  init();
})(Ws);
```
