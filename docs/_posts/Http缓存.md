---
title: Http缓存
date: 2022-04-28 09:27:06
tags:
  - Http
categories:
  - 浏览器
permalink: /pages/4d0228/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

HTTP 缓存分为强缓存和协商缓存。优先级较高的是强缓存，在命中强缓存失败的情况下，才会走协商缓存。

## 强缓存

强缓存指的是向浏览器缓存查找该请求的结果，并根据该结果的缓存规则来决定是否使用该缓存结果的过程

强缓存是利用 http 响应头中的 `Expires` 和 `Cache-Control` 两个字段来控制的。

### Expires

实现强缓存，过去我们一直用 expires。在服务器的响应头里，会将过期时间写入 expires 字段：

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/}G{%UIG76KBR2YM0H0E47OW.png)

那么，当我们试图再次向服务器请求资源时，浏览器就会先对比本地时间和 expires 的时间，如果本地时间小于 expires 设定的过期时间，就直接去缓存中取这个资源。

不过 expires 依赖于本地时间，如果服务端和客户端的时间设置不同，那么 expires 将无法达到我们的预期

### Cache-Control

考虑到 expires 的局限性，HTTP1.1 新增了 Cache-Control 字段来完成 expires 的任务。**当 Cache-Control 与 expires 同时出现时，我们以 Cache-Control 为准。**

Cache-Control 包含以下几个值：

- **max-age**
  > cache-control: max-age=31536000

max-age 会等于一个时间长度（以秒为单位）。在本例中，max-age 是 31536000 秒，它意味着该资源在 31536000 秒以内都是有效的，完美地规避了时间戳带来的潜在问题。

在代理服务器中，我们使用 s-maxage 来执行 max-age 的功能。

- **public 与 private**

如果我们为资源设置了 public，那么它既可以被浏览器缓存，也可以被代理服务器缓存（也就是多个用户可以共享这个缓存）；如果我们设置了 private，则该资源只能被浏览器缓存。

private 为默认值。

但多数情况下，public 并不需要我们手动设置，因为设置了 max-age 就表示响应是可以缓存的。

- **no-store 与 no-cache**

如果设置了 no-cache，则跳过设置强缓存，但是不妨碍设置协商缓存；需要配合 Etag/lastModify 使用。每次请求都回询问服务端。（但是如果存在 max-age，得等 max-age 过期，再走协商缓存）

如果设置了 no-store ，表示严格不缓存，所有内容都不会被缓存，即不使用强制缓存，也不使用协商缓存，每次都要从服务器重新拿资源（就算设置了 max-age，max-age 也不生效， no-store 优先级最高）

## 协商缓存

协商缓存指的是强制缓存失效后(一般是 max-age 或 expires 过期)，浏览器向服务器询问缓存的相关信息，进而判断是重新发起请求还是从本地拿缓存的过程。

如果服务端提示缓存资源未改动（Not Modified），资源会被重定向到浏览器缓存，这种情况下网络请求对应的状态码是 304

同样，协商缓存的标识也是在响应报文的 HTTP 头中和请求结果一起返回给浏览器的，控制协商缓存的字段分别有：`Last-Modified / If-Modified-Since`和`Etag / If-None-Match`，其中 Etag / If-None-Match 的优先级比 Last-Modified / If-Modified-Since 高

- **Last-Modified / If-Modified-Since**

如果我们启用了协商缓存，Last-Modified 会在首次请求时随着响应头返回：

> Last-Modified: Fri, 27 Oct 2017 06:35:57 GMT

随后我们每次请求时，会带上一个叫 If-Modified-Since 的时间戳字段，它的值正是上一次 response 返回给它的 last-modified 值：

> If-Modified-Since: Fri, 27 Oct 2017 06:35:57 GMT

服务器接收到这个时间戳后，会比对该时间戳和资源在服务器上的最后修改时间是否一致，从而判断资源是否发生了变化。如果发生了变化，就会返回一个完整的响应内容，并在响应头中添加新的 Last-Modified 值；否则，返回如上图的 304 响应，响应头不会再添加 Last-Modified 字段,下次浏览器发起请求时，带上的还是旧的 Last-Modified

- **Etag / If-None-Match**

Etag 是由服务器**为每个资源生成的唯一的标识字符串，这个标识字符串是基于文件内容编码的**，只要文件内容不同，它们对应的 Etag 就是不同的。

当首次请求时，我们会在响应头里获取到一个最初的标识符字符串：

> ETag: W/"2a3b-1602480f459"

那么下一次请求时，请求头里就会带上一个值相同的、名为 if-None-Match 的字符串供服务端比对:

> If-None-Match: W/"2a3b-1602480f459"

不过 Etag 的生成过程需要服务器额外付出开销，会影响服务端的性能。

## 怎么设置强缓存与协商缓存

- 后端服务器如 nodejs:

```js
res.setHeader('max-age': '3600 public')
res.setHeader(etag: '5c20abbd-e2e8')
res.setHeader('last-modified': Mon, 24 Dec 2018 09:49:49 GMT)
```

- nginx 配置添加响应头
  ![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220428102038.png)

## 总结

总结一些经验和问题 可能不定时更新

- Expires 和 Cache-Control 字段用来做强缓存 ，Last-Modified / If-Modified-Since 和 Etag / If-None-Match 用来做协商缓存

- expires 和 max-age 可以共存，但是 max-age 优先级高

- no-cache 存在的情况下，如果还配置了 max-age，那么只能等 max-age 过期才能去通过 no-cache 进行下一步的协商缓存请求，所以 max-age 和 no-cache 是可以共存的，且 max-age 的级别是要高于 no-cache 的

- 图解整个缓存过程

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/20220428105915.png)
