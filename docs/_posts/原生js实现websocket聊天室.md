---
title: 原生js实现websocket聊天室
date: 2021-03-17 21:56:28
tags:
  - websocket
categories:
  - websocket
permalink: /pages/754d95/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

## 简介

WebSocket 是一种网络通信协议，很多高级功能都需要它。

初次接触 WebSocket 的人，都会问同样的问题：我们已经有了 HTTP 协议，为什么还需要另一个协议？它能带来什么好处？

答案很简单，因为 HTTP 协议有一个缺陷：通信只能由客户端发起。

举例来说，我们想了解今天的天气，只能是客户端向服务器发出请求，服务器返回查询结果。HTTP 协议做不到服务器主动向客户端推送信息。

这种单向请求的特点，注定了如果服务器有连续的状态变化，客户端要获知就非常麻烦。我们只能使用"轮询"：每隔一段时候，就发出一个询问，了解服务器有没有新的信息。最典型的场景就是聊天室。

轮询的效率低，非常浪费资源（因为必须不停连接，或者 HTTP 连接始终打开）。因此，工程师们一直在思考，有没有更好的方法。WebSocket 就是这样发明的。

WebSocket 协议在 2008 年诞生，2011 年成为国际标准。所有浏览器都已经支持了。

它的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于服务器推送技术的一种。

## websocket 事件

### websocket 事件

在创建聊天室之前，我们需要知道 websocket 有以下事件：

| 事件    | 事件处理程序     | 描述                       |
| ------- | ---------------- | -------------------------- |
| open    | Socket.onopen    | 连接建立时触发             |
| message | Socket.onmessage | 客户端接收服务端数据时触发 |
| error   | Socket.onerror   | 通信发生错误时触发         |
| close   | Socket.onclose   | 连接关闭时触发             |

### 前端用到的事件

前端需要一些事件处理函数的绑定，他们是：

- open
- message
- error
- close

### 后端用到的事件

- open
- message
- error
- close
- connection

### 库

- ws

来，准备好了就开始

## 前端

### 项目构建

我们用`npm init -y`来初始化环境，然后在用`yarn add vite -D`来构建目录

然后再 package.json 中的 script 下添加启动命令：

```json
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "vite"
  },
```

### 基础视图

我们构建两个视图，如下：

#### entry.html -

- Input username
- localStorage to save the username
- click for enter the chatting room

#### index.html -

- list - show messages
- listinput - message
- btn - send message

我们先来建立 entry,并写上内容

在 entry 中，我们要输入用户名，并校验用户名，存到 localstorage 中。然后跳转到 index 页面

### 代码编写

`entry.html`

```html
<!-- entry.html -->
<body>
  <input type="text" id="username" placeholder="请输入用户名" />
  <button id="enter">进入聊天室</button>
  <script src="js/entry.js"></script>
</body>
```

`entry.js`

```js
// entry.js
((doc, storage, location) => {
  const oUsername = doc.querySelector('#username');
  const oEnterBtn = doc.querySelector('#enter');

  const init = () => {
    bindEvent();
  };
  // 给enter绑定事件
  function bindEvent() {
    oEnterBtn.addEventListener('click', handleEnterBtnClick, false);
  }

  function handleEnterBtnClick() {
    const username = oUsername.value.trim();
    // 验证用户名
    if (username.length < 6) {
      alert('用户名不小于6位');
      return;
    }
    // local存储
    storage.setItem('username', username);
    location.href = 'index.html';
  }

  init();
})(document, localStorage, location);
```

`index.html`

```html
<body>
  <ul id="list"></ul>
  <input type="text" id="message" placeholder="请输入消息" />
  <button id="send">发送</button>
  <script src="js/index.js"></script>
</body>
```

`index.js`

```js
((doc, Socket, storage, location) => {
  const oList = doc.querySelector('#list');
  const oMsg = doc.querySelector('#message');
  const oSendBtn = doc.querySelector('#send');
  const ws = new Socket('ws:localhost:8000');
  let username = '';

  const init = () => {
    bindEvent();
  };

  function bindEvent() {
    oSendBtn.addEventListener('click', handleSendBtnClick, false);
    ws.addEventListener('open', handleOpen, false);
    ws.addEventListener('close', handleClose, false);
    ws.addEventListener('error', handleError, false);
    ws.addEventListener('message', handleMessage, false);
  }

  function handleSendBtnClick() {
    const msg = oMsg.value;

    if (!msg.trim().length) {
      return;
    }
    // websocket发送消息给后端，使用json.stringfy方法转为字符串
    ws.send(
      JSON.stringify({
        user: username,
        dateTime: new Date().getTime(),
        message: msg,
      })
    );
    oMsg.value = '';
  }
  // 验证本地是否有用户名，没有返回登录
  function handleOpen(e) {
    console.log('Websocket open', e);
    username = storage.getItem('username');

    if (!username) {
      location.href = 'entry.html';
      return;
    }
  }

  function handleClose(e) {
    console.log('Websocket close', e);
  }

  function handleError(e) {
    console.log('Websocket error', e);
  }
  // 接收后端发送消息
  function handleMessage(e) {
    console.log('Websocket message');
    const msgData = JSON.parse(e.data);
    oList.appendChild(createMsg(msgData));
  }
  //根据消息创建dom节点
  function createMsg(data) {
    const { user, dateTime, message } = data;
    const oItem = doc.createElement('li');
    oItem.innerHTML = `
      <p>
        <span>${user}</span>
        <i>${new Date(dateTime)}</i>
      </p>
      <p>消息：${message}</p>
    `;

    return oItem;
  }

  init();
})(document, WebSocket, localStorage, location);
```

## 后端

后端需要 ws 包，使用 yarn add ws 安装

然后新建 index.js ，里面内容如下:

```js
const Ws = require('ws');

// 立即执行函数
((Ws) => {
  // 建立websocket服务
  // 访问ws:localhost:8000

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
    console.log('Websocket open');
  }

  function handleClose() {
    console.log('Websocket close');
  }

  function handleError() {
    console.log('Websocket error');
  }
  //用户进入聊天室触发事件
  function handleConnection(ws) {
    console.log('Websocket connected');

    ws.on('message', handleMessage);
  }
  //向每个客户端都发送消息
  function handleMessage(msg) {
    server.clients.forEach(function (c) {
      c.send(msg);
    });
  }

  init();
})(Ws);
```

这样，我们的聊天室就搭建成功了

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/websocketConnect.png)

源码地址：

https://github.com/gezhicui/jswebsocket
