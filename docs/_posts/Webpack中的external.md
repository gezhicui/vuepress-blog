---
title: Webpack中的external
date: 2022-05-16 17:23:16
permalink: /pages/465c21/
sidebar: auto
categories:
  - Webpack
tags:
  - Webpack
  - 性能优化
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

在众多的 webpack 配置教程中，对 externals 这个配置选项，总是一带而过，把文档中提到的几种方式都复述一遍，但是对于开发者而言，根本没法完全理解。本文试图通过一整篇文章，详细的对 externals 这个参数进行讲解。

<!-- more -->

## bundle-analyzer 插件

如果小伙伴有做过首屏加载时间优化，应该会遇到 chunk-vendors.js 这个文件，巨大无比，加载时间超长，是首屏加载时间过长的罪魁祸首之一。

下面通过一个实际的项目来演示，先通过插件`webpack-bundle-analyzer`来可视化地查看 chunk-vendors.js 这个文件里面的内容。

> npm install webpack-bundle-analyzer --save-dev

在 vue.config.js 中引入这个插件

```js
const BundleAnalyzerPlugin =
  require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
module.exports = {
  configureWebpack: (config) => {
    return {
      plugins: [new BundleAnalyzerPlugin()],
    };
  },
};
```

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220516172634.png)

在上面的分析图中我们可以看到，最大的两个 js 文件，里面都是一些第三方依赖包，那么只要把这些依赖包提取出来，就可以解决 chunk-vendors.js 过大的问题。下面就使用 externals 来提取这些依赖包，其实应该说用 externals 来防止这些依赖包被打包。

## externals 配置

首先 externals 的值是个对象

```js
module.exports = {
  configureWebpack: (config) => {
    externals: {
      key: value;
    }
  },
};
```

其中 key 是第三方依赖库的名称，同 package.json 文件中的 dependencies 对象的 key 一样。下图红框所示，其中 value 值可以是字符串、数组、对象。

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220516172938.png)

说起 externals，网上已经很多教程了，但是讲解都不是很详细，特别是对 value 值的讲解。如果 value 值不正确导致的报错，也不知道怎么去排查和修复。

按我的理解 value 值应该是**第三方依赖编译打包后生成的 js 文件，然后 js 文件执行后赋值给 window 的全局变量名称。**

那怎么找这个全局变量名称呢，有一个方法

上面所说的 js 文件就是要用 CDN 引入的 js 文件。那么可以通过浏览器打开 CDN 链接。由于代码是压缩过，找个在线 js 格式化把代码处理一下，就可以阅读代码了。

### 提取 element-ui

例如要提取 element-ui 这个依赖包。其 CDN 链接是[ element-ui CDN ](https://unpkg.com/element-ui@2.10.1/lib/index.js),格式化后代码如下，是个自执行函数。

```js
function(e, t) {
    "object" == typeof exports && "object" == typeof module ? //判断环境是否支持commonjs模块规范
    module.exports = t(require("vue")) :
    "function" == typeof define && define.amd ? //判断环境是否支持AMD模块规范
    define("ELEMENT", ["vue"], t) :
    "object" == typeof exports ? //判断环境是否支持CMD模块规范
    exports.ELEMENT = t(require("vue")) :
    e.ELEMENT = t(e.Vue)
} ("undefined" != typeof self ? self: this,function(e){
    //省略...
});
```

在代码块中，有一句`e.ELEMENT = t(e.Vue)`，其中 e 为 window 对象，那么赋值给 window 的全局变量名称是 ELEMENT,所以我们的 externals 的配置如下

```js
module.exports = {
  configureWebpack: {
    externals: {
      'element-ui': 'ELEMENT',
    },
  },
};
```

在 public/index.html 中引入

```html
<body>
  <div id="app"></div>
  <script src="https://unpkg.com/element-ui@2.10.1/lib/index.js"></script>
</body>
```

但是会发现报错了

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220516173455.png)

回过来看 e.ELEMENT = t(e.Vue),发现还需要 Vue。那么把 Vue 依赖包也提取出来。

### 提取 Vue

Vue 编译打包生成 js 文件的 CDN 链接是[ Vue CDN ](https://cdn.bootcdn.net/ajax/libs/vue/2.6.11/vue.runtime.min.js) 格式化后代码如下，是个自执行函数。

```js
function(t, e) {
    "object" == typeof exports && "undefined" != typeof module ?
    module.exports = e() :
    "function" == typeof define && define.amd ?
    define(e) :
    (t = t || self).Vue = e()
} (this,function(){
    //省略...
})
```

在浏览器中最后执行`(t = t || self).Vue = e()`，其中 t 为 this，this 是 window 对象。那么赋值给 window 的全局变量名称是 Vue。 在 vue.config.js 中配置如下

```js
module.exports = {
  configureWebpack: {
    externals: {
      'element-ui': 'ELEMENT',
      vue: 'Vue',
    },
  },
};
```

在 public/index.html 中引入

```html
<body>
  <div id="app"></div>
  <script src="https://unpkg.com/element-ui@2.10.1/lib/index.js"></script>
  <script src="https://cdn.bootcdn.net/ajax/libs/vue/2.6.11/vue.runtime.min.js"></script>
</body>
```

刷新一下，发现还是报错 555

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220516173737.png)

回到 public/index.html，vue 在 element-ui 后面引入，难怪会报错，vue 一定要在 element-ui 之前引入，修改一下

```html
<body>
  <div id="app"></div>
  <script src="https://cdn.bootcdn.net/ajax/libs/vue/2.6.11/vue.runtime.min.js"></script>
  <script src="https://unpkg.com/element-ui@2.10.1/lib/index.js"></script>
</body>
```

刷新页面 没有报错了，现在我们知道 value 值是从哪里来的。遇到错误知道排查和修复。

### 提取 vue-router

接下来继续把 vue-router 依赖和 xlsx 依赖提取了
vue-router 编译打包生成 js 文件的 CDN 链接是[ Vue-router CDN ](https://cdn.bootcdn.net/ajax/libs/vue-router/3.2.0/vue-router.min.js)

格式化后代码如下:

```js
var t, e;
(t = this),
  (e = function () {
    //省略...
  }),
  'object' == typeof exports && 'undefined' != typeof module
    ? (module.exports = e())
    : 'function' == typeof define && define.amd
    ? define(e)
    : ((t = t || self).VueRouter = e());
```

在浏览器中最后执行`((t = t || self).VueRouter = e()`，其中 t 为 this，this 是 window 对象。那么赋值给 window 的全局变量名称是 VueRouter。 在 vue.config.js 中配置如下

```js
module.exports = {
  configureWebpack: {
    externals: {
      'element-ui': 'ELEMENT',
      vue: 'Vue',
      'vue-router': 'VueRouter',
    },
  },
};
```

在 public/index.html 中引入

```html
<body>
  <div id="app"></div>
  <script src="https://cdn.bootcdn.net/ajax/libs/vue/2.6.11/vue.runtime.min.js"></script>
  <script src="https://cdn.bootcdn.net/ajax/libs/vue-router/3.2.0/vue-router.min.js"></script>
  <script src="https://unpkg.com/element-ui@2.10.1/lib/index.js"></script>
</body>
```

重新 npm run dev 后刷新页面，正常无报错，继续提取 xlsx 依赖包。

### 提取 xlsx

xlsx 编译打包生成 js 文件的 CDN 链接是[ xlsx CDN ](https://cdn.bootcdn.net/ajax/libs/xlsx/0.16.1/xlsx.min.js)， 格式化后代码如下。

```js
var XLSX = {};
function make_xlsx_lib(e){
    //省略...
}
if (typeof exports !== "undefined") make_xlsx_lib(exports);
else if (typeof module !== "undefined" && module.exports) make_xlsx_lib(module.exports);
else if (typeof define === "function" && define.amd) define(function() {
    if (!XLSX.version) make_xlsx_lib(XLSX);
    return XLSX
});
else make_xlsx_lib(XLSX);
var XLS = XLSX,
```

发现这下子不好判断，赋值给 window 的全局变量名称是那个了。先猜测是 XLSX， 在 public/index.html 中引入

```html
<body>
  <div id="app"></div>
  <script src="https://cdn.bootcdn.net/ajax/libs/vue/2.6.11/vue.runtime.min.js"></script>
  <script src="https://cdn.bootcdn.net/ajax/libs/vue-router/3.2.0/vue-router.min.js"></script>
  <script src="https://unpkg.com/element-ui@2.10.1/lib/index.js"></script>
  <script src="https://cdn.bootcdn.net/ajax/libs/xlsx/0.16.1/xlsx.min.js"></script>
</body>
```

把 public/index.html 这个文件丢到浏览器打开，然后在控制台输入`window.XLSX`，看看有没有值。

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220517110639.png)

有值不为 undefined，说明赋值给 window 的全局变量名称是 XLSX。那么在 vue.config.js 中配置如下

```js
module.exports = {
  configureWebpack: {
    externals: {
      'element-ui': 'ELEMENT',
      vue: 'Vue',
      'vue-router': 'VueRouter',
      xlsx: 'XLSX',
    },
  },
};
```

重新 npm run dev 后刷新页面，正常无报错。

## 优化完成总结

用 externals 提取完第三依赖包后，执行 npm run build，分析图如下

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220517111035.png)

和之前对比，两个大文件的一个已经消失，剩下一个只有 81.42KB，和之前的 1003.44KB 相比是不是小了很多。再看里面的内容，之前的 elemen-ui、jquery.js、vue.runtime.esm.js、xlsx.js 这些依赖包都消失。优化成效明显。
