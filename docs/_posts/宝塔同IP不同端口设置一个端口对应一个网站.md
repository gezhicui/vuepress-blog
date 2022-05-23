---
title: 宝塔同IP不同端口设置一个端口对应一个网站
date: 2021-02-17 10:39:00
tags:
  - 服务器搭建
categories:
  - 部署
permalink: /pages/79dc41/
sidebar: auto
author:
  name: 杨雨翔
  link: https://gitee.com/xiang0515
---

当我们想在宝塔下添加了一个站点之后（默认端口为 80，如：139.224.211.111:3001），要再创建一个新站点时（如：139.224.211.111:3002）就会报错：

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/baotaerror.png)

我的宝塔环境是 Linux+PHP+Nginx+Mysql；

## 一、准备工作：

- IP 一个，例如：192.168.1.666；
- 服务器一台，放行所需端口；
- 假想一个域名 www.test.com；
- 宝塔面板；

## 实现效果：

192.168.1.666:6601 访问站点 1 ；

192.168.1.666:6602 访问站点 2；

## 实现步骤

1、新建一个站点，比如： www.test.com
2、修改当前站点的配置文件

```
listen 80;
server_name  www.test.com;
```

修改为

```
listen 3001;
server_name 192.168.1.666;
```

3、修改域名管理

添加一个

192.168.1.666：6601

4、把之前添加那个，www.test.com 删除 ！

总结：照此原理，可再添加多个。切记，改完之后，一定要重启 nginx 再访问，即可看到站点内容！
