---
title: rem布局
date: 2020-10-04 10:56:20
tags:
  - css
categories:
  - css
permalink: /pages/fcb0e0/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

## 常见移动 web 布局适配方法

- 固定高度，宽度百分比：这种方法只适合简单要求不高的 webApp，几乎达不到大型项目的要求，属于过时的方法。
- Media Query（媒体查询）：现在比较主流的适配方案，比如我们常用的样式框架 Bootstrap 就是靠这个起家的，它能完成大部分项目需求，但是编写过于复杂。
- flex 布局：主流的布局方式，不仅适用于移动 Web，网页上也表现良好，这也是现在工作中用的最多的布局方式，那我们的项目尽量采用 flex+rem 的方式进行布局和完成移动端的适配。

## rem 单位介绍

&emsp;&emsp;rem（font size of the root element）是相对长度单位。相对于根元素（即 html 元素）font-size 计算值的倍数。
&emsp;&emsp;适配原理：将 px 替换成 rem，动态修改 html 的 font-size 适配。它可以很好的根据根元素的字体大小来进行变化，从而达到各种屏幕基本一直的效果体验。
现在我们作一个实验，你可以新建一个网页，并写入如下代码：

```html
<div class="test">
  <p class="hello">Hello</p>
</div>
```

&emsp;&emsp;然后给 html 一个基本的样式:

```css
.test {
  width: 320px;
  height: 160px;
  background-color: bisque;
  text-align: text；;
}
.hello {
  color: red;
}
```

&emsp;&emsp;上边我们使用了还是传统的使用 px 作为单位，我们在移动端调试模式 iphone5 环境查看一下。会发现 div 的宽度是正好的，我们再查看一下字体，发现大小是 16px。

&emsp;&emsp;我们现在可以把 CSS 中的 px 单位换成 rem 单位来进行测试。因为 html 根元素的字体大小是 16px，那么换成 rem 单位，直接除以 16 就好。

```css
.test {
  width: 20rem;
  height: 10rem;
  background-color: blue;
  text-align: text；;
}
.hello {
  color: red;
  font-size: 1rem;
}
```

&emsp;&emsp;页面并没有什么变化，也就是说我们掌握了换算关系。为了更好的说明这点，我们可以试着给 html 根样式加入字体大小，比如换成 font-size:32px;。这时页面和字体都扩大了一倍。但是我们现在还是不能实现适配，因为我们根元素的字体是固定的。

## JS 控制适配屏幕

&emsp;&emsp;明白了 REM 的原理后，我们就可以使用这个特点来进行适应布局了，这也是现在比较主流的移动端 web 适配方案。 三行 JS 代码完成适配：

```js
//得到手机屏幕的宽度
let htmlWidth =
  document.documentElement.clientWidth || document.body.clientWidth;
//得到html的Dom元素
let htmlDom = document.getElementsByTagName('html')[0];
//设置根元素字体大小
htmlDom.style.fontSize = htmlWidth / 20 + 'px';
```

搞定啦

tips：补充个小知识。在进行移动端布局的时候，在`meta`属性的`content`中加入`user-scalable=no`,即禁止用户缩放
