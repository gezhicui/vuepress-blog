---
title: canvas生成验证码
date: 2021-02-16 21:15:59
tags:
  - canvas
categories:
  - JavaScript
permalink: /pages/b69126/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

首先先生成 canvas 基本结构：

```html
< ! DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title></title>
  </head>
  <body>
    <canvas id="canvas" width="120" height="40"></canvas>
  </body>
</html>
```

这里就生成了一个 id 为 canvas 的画布，然后我们来解决 js

首先，我们要有随机生成数字和颜色的方法

```js
//随机数的生成函数
function rn(min, max) {
  return parseInt(Math.random() * (max - min) + min);
}
//随机生成颜色的函数
function rc(min, max) {
  var r = rn(min, max);
  var g = rn(min, max);
  var b = rn(min, max);
  return 'rgb(${r},${g},$(b))';
}
```

然后 我们来初始化画布：

```js
// 设置宽高
var w = 120;
var h = 40;
// 获取画布对象
var canvas = document.querySelector('#canvas');
// 获取2d画笔
var ctx = canvas.getContext('2d');
```

初始化完，我们就可以开始画了

```js
// 首先，填充一下随机颜色  这里用180-230这个浅色范围
ctx.fillstyle = rc(180, 230);
// 填充矩形
ctx.fillRect(0, 0, w, h);
```

到这里，我们就实现了一个有颜色的画布区域，然后开始绘制随机字符串

```js
//随机字符串
var pool = 'ABCDEFGHIGKLIMNOPQRSTUVwXYZ1234567890';
for (var i = 0; i < 4; i++) {
  //取出随机的字母或数字 ，到时候定义一个string+=，返回回去就行
  var c = pool[rn(0, pool.length)];
  //随机出字体大小
  var fs = rn(18, 40);
  //随机字母数字的旋转角度
  var deg = rn(-30, 30);
  ctx.font = fs + 'px Simhei ';
  //设置或返回在绘制文本时使用的当前文本基线。
  ctx.textBaseline = 'top ';
  //设置字体的填充颜色
  ctx.fillStyle = rc(80, 150);
  // 保存当前画笔样式
  ctx.save();
  // 设置当前i的字符的位置
  ctx.translate(30 * i + 15, 15);
  // 随机略微旋转
  ctx.rotate((deg * Math.PI) / 180);
  // 填充，后面两个参数可以自行调整位置
  ctx.fillText(c, 0, 0);
  // 恢复到保存之前状态，不然旋转会出问题
  ctx.restore();
}
```

到这里，我们已经可以实现了基本的验证码的样式

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/yanzhenma.png)

接下来，我们就随机生成一些干扰的线，我们在画布范围内随机生成 5 条线段

```js
//随机生成5条干扰线
for(var i=;i<5;i++){
  ctx.beginPath()
  ctx.moveTo(rn(0,w) ,rn(0,h));
  ctx.lineTo( rn(0,w) ,rn(0,h));
  ctx.strokeStyle = rc( 180,230);
  ctx.closePath();
  ctx.stroke()
}
```

然后，我们得到了这样的效果：

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/ganraoxian.png)

那我们现在还不够，还想要做一些小圆点进行干扰，那就继续加:

```js
//随机产生40个小的干扰圆点
for (var i = o; i < 40; i++) {
  ctx.beginPath();
  // x y 半径 0-2pi的圆
  ctx.arc(rn(0, w), rn(0, h), 1, 0, 2 * Math.PI);
  ctx.closePath();
  ctx.fillStyle = rc(150, 200);
  ctx.fill();
}
```

这样，我们就已经能生成随机小圆点了

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/suijixiaoyuandian.png)

然后把他自定义封装成一个函数，就大功告成！
