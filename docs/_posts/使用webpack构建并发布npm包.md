---
title: 使用webpack构建并发布npm包
date: 2022-07-27 16:08:16
sidebar: auto
categories: 
  - Webpack
tags: 
  - Webpack
author: 
  name: 杨雨翔
  link: https://github.com/gezhicui
permalink: /pages/3082d3/
---

由于业务中经常使用到省市列表的树结构，经常对树结构进行各种各样的转换，所以想封装一个工具包，一劳永逸。在制作真正的包前肯定要先写个玩具包测试可行性，同时测试使用`umd`的模式来打包，兼容`amd、cjs、esm`规范的情况。所以就有了`test-npm-publish-yyx`这个测试包，顺便记录一下 `npm` 工具包的整个开发流程

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220727161227.png)

<!-- more -->

## 开发目标

在开始之前，我给这个玩具包定了以下几个目标：

- 实现两数加法功能
- 支持压缩打包和非压缩打包
- 支持 esm、cjs、amd 模块引入

开始实践！

## 创建项目并实现功能

```bash
# 创建文件夹
mkdir test-npm-publish-yyx
# 创建package.json
npm init -y
# 安装webpack
npm i webpack webpack-cli -D
```

然后创建以下目录结构

```bash
-src
  -addNumber.js
-index.js
-webpack.config.js
```

在`src/addNumber.js`中添加以下代码

```js
export default function add(a, b) {
  return a + b;
}
```

以上代码实现了最简单的两数之和方法，接下来配置 webpack，在`webpack.config.js`添加以下配置：

```js
const TerserPlugin = require('terser-webpack-plugin');

module.exports = {
  mode: 'none',
  entry: {
    'add-number': './src/addNumber.js',
    'add-number.min': '/src/addNumber.js',
  },
  output: {
    filename: '[name].js',
    globalObject: 'this',
    libraryTarget: 'umd', //支持库的引入方式
    libraryExport: 'default', //默认导出
  },
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        //此插件在webpack4之后，当mode 设置为production时，默认开启压缩
        include: /\.min\.js$/, //匹配min.js结尾的文件进行压缩
      }),
    ],
  },
};
```

然后，就可以在`package.json`里添加打包命令和外部访问的入口文件了

```diff
{
+ "main": "index.js",
  "scripts": {
+   "build": "npx webpack --config webpack.config.js"
  },
}
```

`index.js`的内容是通过判断环境变量实现不同文件的导出，代码如下

```js
if (process.env.NODE_ENV === 'production') {
  module.exports = require('./dist/add-number.min.js');
} else {
  module.exports = require('./dist/add-number.js');
}
```

全都配置完毕后，就可以通过`npm run build`进行打包了，打包产物如下：

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220727163739.png)

## 创建 npm 账号并部署包到 npm 官网上

想要在 `npm` 发布包，我们首先需要在 `npm` 注册一个账号,这个就不赘述了，百度一下就有

::: tip
发布的 npm 包的名称是在 package.json 中的 name,与本地目录文件名无关，npm 包名与现有包名冲突，会发包失败，发包前先在 npm 上面找找看有没有同名的包
:::

注册成功之后需要在自己的电脑本地终端输入`npm login`进行 npm 登录，**首次需要登录,npm login 存储证书到本地,后面就不需要每次都登录的**

登录完成后，输入`npm publish`就可以部署你的包了，每次`publish`时需要修改`package.json`中的`version`,更新下版本，发相同版本的包也会失败

## 测试

发包完成，在`vue`项目中和`node`下都下载包测试一下看看能不能用

### Vue 环境测试

vue 环境使用 esm 导入包，先下载包

```bash
npm install test-npm-publish-yyx
```

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220727164939.png)

在浏览器控制台中查看，发现打印两数之和了，可以用

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220727165027.png)

### Node 环境测试

一样，在 node 项目中先下载包

```bash
npm install test-npm-publish-yyx
```

然后通过 cjs 引入

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220727165148.png)

执行一下`app.js`

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220727165213.png)

大功告成！根据以上步骤发现包是可用的，接下来就是对包的功能进行拓展了，先在`npm`占了个名叫`tree-formatter`的坑位，后续就在这个包上对想要的方法进行开发
