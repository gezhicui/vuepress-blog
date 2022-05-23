---
title: Nodejs中的密码加密bcrypt
date: 2020-07-22 17:08:24
tags:
  - Node.js
  - 密码加密
categories:
  - Node.js
permalink: /pages/b38d22/
sidebar: auto
author:
  name: 杨雨翔
  link: https://gitee.com/xiang0515
---

## 密码加密 bcrypt

&emsp;&emsp;哈希加密是单程加密方式:1234 => abcd，无法解密。但是，在加密的密码中加入随机字符串可以增加密码被破解的难度。

先看一个小 demo 介绍

```js
//导入bcrypt模块
const bcrypt = require ('bcrypt') ;
    //生成随机字符串
    // genSalt方法接收一个数值作为参数
    //数值越大生成的随机字符串复杂度越高
    //数值越小生成的随机字符串复杂度越低
    //默认值是10
    //返回生成的随机字符串
let salt = await bcrypt.gensalt(10);
    //对密码进行加密
    //第一个参数．要进行加密的明文
    //第二个参数．随机字符串
    //返回值是加密后的密码
let pass = await bcrypt.hash ('明文密码,'salt);

//密码比对
let isEqual = await bcrypt.compare('明文密码','加密密码');
```

## bcrypt 安装

> npm install bcrypt

### bcrypt 依赖的其他环境(需要提前安装)

- python 2.x
  > 去官网安装
- node-gyp
  > npm install -g node-gyp
- windows-build-tools
  > npm install --global --production windows-build-tools

## bcrypt 的使用

```js
//在model下的user.js中
//创建用户集合
//引入mongoose第三方模块
const mongoose = require('mongoose');
//导入bcrypt
const bcrypt = require('bcrypt');
//创建用户集合规则
const userSchema = new mongoose.Schema({
  username: String,
  email: String,
  password: String,
});
//创建集合
const User = mongoose.model('User', userSchema);

async function run() {
  const salt = await bcrypt.genSalt(10);
  const pass = await bcrypt.hash('123456', salt);
  const user = await User.create({
    username: 'yang',
    email: 'yang@qq.cn',
    password: pass,
    role: 'admin',
    state: 0,
  });
}
creatUser();
```

### 在验证登录逻辑中加上密码的比对

```js
//实现登录功能
admin.post('/login', async(req,res）=>{
    //接收请求参数 解构
    const {email,password} = req.body;
    //如果用户没有输入邮件地址
    if (email.trim().length == 0 || password.trim().length = 0) return res.status(400).send('请输入邮件地址或密码')
    //根据邮箱地址查询用户信息
    //如果查询到了用户user变量的值是对象类型对象中存储的是用户信息
    //如果没有查询到用户user变量为空
    let user = await User.findOne({email});
    //查询到了用户
    if(user){
        //将客户端传递过来的密码和用户信息中的密码进行比对
        // true比对成功
        // false对比失败
        let isValid = await bcrypt.compare(password,user.password)
        if(isValid){
            //比对成功
            res.send('登录成功')
        }else{
            //密码不一样
            res.status(400).render('admin/error', {msg:'邮箱地址或者密码错误'})
        }
    //没查到
    }else{
        res.status(400).render('admin/error', {msg:'邮箱地址或者密码错误'})
    }
}
```

### 密码加密传入数据库

```js
//对密码进行加密处理
//生成随机字符串
const salt = await bcrypt.genSalt(10);
//加密
//替换密码
req.body.password = password;
//将用户信息添加到数据库中,用户信息就是post请求传过来的req.body中的数据
await User.create(req.body);
//将页面重定向到用户列表页面
res.redirect('/admin/user');
const password = await bcrypt.hash(req.body.password, salt);
```
