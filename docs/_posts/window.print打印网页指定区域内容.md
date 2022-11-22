---
title: window.print打印网页指定区域内容
date: 2022-11-22 21:28:19
tags:
  - JavaScript
categories:
  - JavaScript
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
permalink: /pages/f55937/
---

`window.print`方法用于打印当前窗口的内容。但是最近有个需求是点击按钮出现一个弹出框，只需要打印弹出框里的内容，如下图:

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/lQLPJxbf7kuTqynNA7nNBQCwGutHebDDHHADb_RgqADOAA_1280_953.png)

只需要打印出弹出框里面的内容，这里记录一下实现方式

<!-- more -->

要想打印出在指定区域的内容，那肯定要先找到区域在哪里，我们把弹出框中的内容用一个 div 包裹起来，取个 id,我这里就叫`printModalContent`

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20221122214051.png)

现在，我们就知道了要打印的是 id 为`printModalContent`的区域的内容,那我们就根据内容区域的`innterHTML`实现获取指定区域的内容

```js
//导入printHtml方法
import printHtml from './print';

//打印按钮调用handlePrint方法
const handlePrint = () => {
  const target = document.getElementById('printModalContent');
  if (target) {
    printHtml(target.innerHTML);
  }
};
```

现在把获取到的内容传入`printHtml`方法中,最重要的就是这个`printHtml`方法的实现，下面就来实现下这个方法，用法在注释里写的明明白白了

```js
//print.js
// 设置打印样式,在这可以自定义打印出来的内容的样式
function getStyle() {
  const styleContent = `#print-container {
    display: none
  }
  @media print {
    body > :not(.print-container) {
        display: none
    }
    html,
    body {
        display: block !important;
    }
    #print-container {
        display: block;
        overflow:visible
    }
    @page {
      margin:5mm 20mm;
    }
  }`;
  const style = document.createElement('style');
  style.innerHTML = styleContent;
  return style;
}

// 清空打印内容
function cleanPrint() {
  const div = document.getElementById('print-container');
  if (div) {
    document.querySelector('body').removeChild(div);
  }
}

// 新建DOM，将需要打印的内容填充到DOM
function getContainer(html) {
  cleanPrint();
  const container = document.createElement('div');
  container.setAttribute('id', 'print-container');
  container.innerHTML = html;
  return container;
}

// 图片完全加载后再调用打印方法
function getLoadPromise(dom) {
  let imgs = dom.querySelectorAll('img');
  imgs = [].slice.call(imgs);

  if (imgs.length === 0) {
    return Promise.resolve();
  }

  let finishedCount = 0;
  return new Promise(resolve => {
    function check() {
      finishedCount += 1;
      if (finishedCount === imgs.length) {
        resolve();
      }
    }
    imgs.forEach(img => {
      img.addEventListener('load', check);
      img.addEventListener('error', check);
    });
  });
}

export default function printHtml(html) {
  const style = getStyle();
  const container = getContainer(html);
  document.body.appendChild(style);
  document.body.appendChild(container);
  getLoadPromise(container).then(() => {
    window.print();
    document.body.removeChild(style);
    document.body.removeChild(container);
  });
}
```

大功告成 打印预览的内容就是这样的：

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/1668483204764_278A87D2-15C2-490d-9935-D39939F1CE0C.png)
