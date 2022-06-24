---
title: FileReader对象处理文件下载失败
date: 2022-06-23 21:04:22
tags:
  - JavaScript
  - FileReader
categories:
  - JavaScript
permalink: /pages/1ac16a/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

在需求开发中，经常遇到文件导出或者文件下载的需求，但是这会涉及到一个问题：后端处理失败，返回的也是二进制流的错误信息，那前端怎么进行失败提示？

记得最早之前我是在响应拦截器中获取到`response`对象进行处理,关于`response`对象，可以查看 MDN:【[Fetch API : Response](https://developer.mozilla.org/zh-CN/docs/Web/API/Response)】。`fetch`请求收到`response`后，由于`response`只能被读取一次，所以我通过` response.clone()`创建响应对象的克隆,**先把对象进行 JSON 解析，如果解析失败，说明返回的不是 json，那就是个二进制流，catch 到这个错误，在错误处理中拿到第二份拷贝进行 blob 解析**，关键代码如下

<!-- more -->

```js
// 响应拦截
async function responseInterceptors(response) {
  //把response拷贝两份，因为response解析完就不能再解析了
  const jsonData = response.clone();
  const blobData = response.clone();
  let result;
  try {
    result = await jsonData.json();
    //-----下面是业务代码------
    const { code, msg } = result;
    if (code && code !== 200) {
      if (code === 401) {
        notification.warning({
          message: '系统提示',
          description: '登录状态已过期，请重新登录',
        });
        history.push({ pathname: '/user/login' });
        localStorage.removeItem('token');
      } else {
        message.error(msg || '请求失败');
      }
    }
    //-----------------------
  } catch {
    console.log('进入blob生成');
    result = await blobData.blob();
  }
  return result;
}
```

能用吗？能。好用吗？不一定。会有两个问题

- 1、在下载文件失败时，由于前端的请求头带上了`responseType: 'blob'`,**会导致即使后端处理失败，返回的错误信息也是`blob`类型，不能直接在页面上提示出来**
- 2、这种方式默认把所有**无法解析成 JSON 的返回结果全丢给 blob 解析**,在业务代码中`blob`就直接转成文件输出了,但是如果遇到 JSON 无法解析,但是也不是正确的 blob 时（虽然没有遇到过这种情况）,下载出来的文件打开就会是乱码,十分不友好

最近总结了使用`FileReader`对象进行`blob`解析，来改善上面说的问题

首先先来发起一个请求到`exportFile`接口

```js
const res = await exportFile(params);
if (res) {
  formatBlobToObject(res, `filename.xls`);
}
```

上面这段代码再熟悉不过了，关键在于`formatBlobToObject`方法的实现，方法实现的关键又在于`FileReader`,可以查看 MDN:【[FileReader](https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader)】

```js
const formatBlobToObject = (res: any, fileName: string) => {
  //判断响应头，如果响应头是application/json，就把blob解析成字符串，再转成json
  if (res?.type === 'application/json') {
    const reader = new FileReader();
    reader.readAsText(res, 'utf-8');
    reader.onload = () => {
      const { resultMsg } = JSON.parse((reader as any)?.result);
      message.error(resultMsg);
    };
  } else {
    // 否则  直接进行文件打印
    saveBlobFile(res, fileName);
  }
};
```

`saveBlobFile`就是我们通常实现的文件下载方法了，也贴在下面

```js
const saveBlobFile = (data: any, filename: string) => {
  if (typeof (window as any)?.chrome !== 'undefined') { // chrome
    const link = document.createElement('a');
    link.href = window.URL.createObjectURL(new Blob([data]));
    link.download = filename;
    link.click();
  } else if (typeof (window.navigator as any)?.msSaveBlob !== 'undefined') { // ie
    const blob = new Blob([data], {
      type: 'application/force-download',
    });
    (window.navigator as any)?.msSaveBlob(blob, filename);
  } else { // firefox
    const file = new File([data], filename, {
      type: 'application/force-download',
    });
    window.open(URL.createObjectURL(file));
  }
};

```

这个方法只要和后端协商好，如果出错，就把响应头设置成`'application/json'`,然后返回错误信息供前端解析就行，如果没出错就进行常规的文件下载，成功解决对 `response` 进行解析存在的问题！nice
