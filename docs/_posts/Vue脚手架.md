---
title: Vue脚手架
date: 2020-06-12 21:35:57
tags:
  - Vue
categories:
  - 框架

permalink: /pages/f808a2/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

## VueCLI

CLI 是什么意思?

&emsp;&emsp;CLI 是 Command-Line Interface,翻译为命令行界面,但是俗称脚手架.Vue CLI 是一个官方发布 vue.js 项目脚手架
&emsp;&emsp;使用 vue-cli 可以快速搭建 Vue 开发环境以及对应的 webpack 配置.

**！VueCLI 使用前必须安装 node 环境和 webpack**

### node 安装

可以直接在官方网站中下载安装。网址: http://nodejs.cn/download/

### webpack 安装

> npm install webpack -g

### VueCLI 的安装

安装 vue 脚手架：(全局安装)

> npm install -g @vue/cli

## VueCLI 的使用

初始化项目可以用两个命令：

> vue create 项目名称
> vue ui 进入图形化界面

### create 创建过程

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/脚手架create.png)

### 创建项目的文件目录

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/脚手架目录结构.png)

## webpack 的配置

&emsp;&emsp;由于 vue-cli3 将 webpack 的基础配置全部内嵌了，这就导致我们初始化项目完成之后发现原先的`webpack`的`config`配置全部都消失不见了，那该怎么办呢？别慌，vue-cli 早就考虑到了这一点，它预留了一个`vue.config.js`的 js 文件供我们对 webpack 进行自定义配置。

### 在项目根目录下新建 vue.config.js 文件与 package.json 同级

&emsp;&emsp;这里的`vue.config.js`会和脚手架创建的`webpack`合并，成为最终配置
