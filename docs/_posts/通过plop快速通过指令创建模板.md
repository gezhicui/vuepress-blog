---
title: 通过plop快速通过指令创建模板
date: 2022-08-03 09:43:43
tags: 
  - 框架
categories: 
  - 框架
sidebar: auto
author: 
  name: 杨雨翔
  link: https://github.com/gezhicui
permalink: /pages/e983c9/
---

在公司的项目中，很多需求是报表开发，但是报表页面间的相似度很高，同事就说能不能提高下开发效率，想个办法**解决重复的工作**，然后我就去找了下解决方案，找到了通过指令创建模板的神器--`plop`，在尝试之后发现可行，于是总结下使用方法

<!-- more -->

## 安装

`plop`是一个微型的脚手架工具，它的特点是可以**根据一个模板文件批量的生成文本或者代码，不再需要手动复制粘贴**，省事省力

前端工程可以通过安装依赖获取 `plop` 的能力

```
// 全局安装
npm i -g plop

// 本地安装
npm i --save-dev plop
```

## 使用

### package.json 配置

安装完依赖，在`package.json`中配置以下命令

```json
  "scripts": {
    "plop": "plop"
  },
```

这样，就可以通过`npm run plop` 来使用`plop`了

### 编写模板

我们先写一个要生成的模板的样子，在`src`下新建一个文件夹`template`(随便建在哪都可以)，然后在文件夹下新增文件`component.hbs`

`plop`使用`Handlebars`作为模板引擎，模板引擎能处理动态内容，在后面的用户自定义输入内容会用到

```js
// src/template/component.hbs
import React, { useEffect, useState } from 'react';

export default () => {

  useEffect(() => {
  }, []);

  return (
    <div>
      {{#if firstBtn}}
        <button>第一个btn</button>
      {{/if}}
      {{#if secondBtn}}
        <button>第二个btn</button>
      {{/if}}
    </div>
  );
};

```

### 编写入口文件

使用`plop`需要一个入口文件,在根目录创建`plopfile.js`,当我们运行`plop`命令的时候，会以这个文件作为入口文件，这个文件需要导出一个函数

```js
module.exports = (plop) => {};
```

这个函数接收到了`plop`,我们可以通过`plop`的`setGenerator`来创建模板，此函数接受两个参数，第一个参数为 `Generator` 的名称,第二个参数是配置项

```js
module.exports = (plop) => {
  plop.setGenerator('component', {
    // 生成器描述
    description: '创建一个组件',
    // 询问用户的问题
    prompts: [],
    // 询问完成后获取到用户输入的内容执行的动作
    actions: (data) => {
      // 需要执行的动作
      const actions = [];
      return actions;
    },
  });
};
```

### prompts

`prompts`配置项是一个数组，会根据数组中的顺序在终端让用户输入问题，比如我们把`prompts`配置成以下内容：

```js
prompts: [
  {
    // 交互类型 input,number,confirm,list,rawlist,expand,checkbox,password,editor
    type: 'input',
    // 变量名，存储用户输入的信息，在后面会用到
    name: 'name',
    // 问题
    message: '请输入组件名称',
    // 输入项验证，返回文本则在终端提示用户文本内容,返回true则通过验证进行下一步
    validate(val) {
      if (!val) {
        return '组件名称不可为空'
      }
      return true;
    },
    // 用户未输入的默认值
    default:'testComponment',
    //当满足函数内的条件时，当前问题才能出现，返回值是Boolean值，函数的参数是之前所有会话的答案
    // when:(answers)=>{
    //   return true/false
    // }
  },
  {
    type: 'confirm',
    name: 'firstBtn',
    default: true,
    message: '组件是否需要第一个按钮?',
    when:(answers)=>{
      // 当组件名不等于test时，询问组件是否需要第一个按钮
      return answers.name!=='test'
    }
  },
  {
    type: 'confirm',
    name: 'secondBtn',
    default: true,
    message: '组件是否需要第二个按钮?'
  },
],
```

配置完问题，就可以在终端输入`npm run plop` 查看配置效果了

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220803103543.png)

接收到用户输入的内容后，就可以在`actions`中执行操作

### actions

`actions`是一个函数，可以接收到`prompts`中用户输入的内容，返回一个数组，数组中包括了要执行的内容

```js
actions: (data) => {
  // 用户输入内容都在data中，可用于js判断操作
  const { name, firstBtn, secondBtn } = data;
  const actions = [];
  actions.push({
    // type string 动作的类型 add(新增文件) append(在某个位置插入) modify(修改文件)
    type: 'add',
    // 操作的路径
    path: './src/pages/{{properCase name}}/index.jsx',
    // 文件模板路径 （内容比较少时，可直接用template）
    templateFile: 'src/template/component.hbs',
  });

  //可以根据data的不同输入给actions push不同的操作，如更新路由文件：
  if (新增个选项询问是否更新路由，判断用户是否需要更新路由) {
    actions.push({
      type: 'append',
      pattern: /(?=(\/\/ flag))/, //正则匹配 //flag,在前面插入
      path: './src/router.js',
      template: `{
            // 测试报表
            name: '测试报表',
            path: '/{{name}}',
            component: './src/pages/{{properCase name}}/index.jsx',
          },\n`,
    });
  }

  return actions;
};
```

其中 `properCase` 将 `dir` 转为大驼峰的格式，常用的关键词：

- `camelCase`: changeFormatToThis 小驼峰
- `properCase`: ChangeFormatToThis 大驼峰
- `snakeCase`: change_format_to_this 下划线分割

我们在`actions`里只用到了`name`属性，`firstBtn, secondBtn`会自动传入`component.hbs`模板文件中进行处理，在`firstBtn`选择`yes`,`secondBtn`选择`no`的情况下，生成的模板文件是这样：

```js
import React, { useEffect, useState } from 'react';

export default () => {
  useEffect(() => {}, []);

  return (
    <div>
      <button>第一个btn</button>
    </div>
  );
};
```

## 多模板

一个项目中可能有多套模板，每套模板有不同的处理逻辑，其实很简单，只要多增加一个`plop.setGenerator`配置就行了，比如

```js
module.exports = (plop) => {
  plop.setGenerator('component', {
    description: '创建一个组件',
    prompts: [],
    actions: (data) => {
      const actions = [];
      return actions;
    },
  });
  plop.setGenerator('report', {
    description: '创建一个报表',
    prompts: [],
    actions: (data) => {
      const actions = [];
      return actions;
    },
  });
};
```

这时候运行`npm run plop`,就会出现多模板选择。

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220803112327.png)

配置单个`setGenerator`时不会出现，是因为只有一个生成器时默认使用该生成器
