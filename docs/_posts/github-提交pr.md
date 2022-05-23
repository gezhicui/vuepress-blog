---
title: github-提交pr
date: 2021-03-09 22:10:58
tags:
  - git
categories:
  - git
permalink: /pages/ddc0a3/
sidebar: auto
author:
  name: 杨雨翔
  link: https://gitee.com/xiang0515
---

在开始之前，确保你有基础的 git 知识

## fork

将项目 fork 到自己的仓库中，可以在 github 的首页搜索到自己的想要的开源项目，我以 ant-design 为例：

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/fork2.png)

我们进入 antd 的 github 页面，点击`fork`,稍等片刻，此项目便会出现在自己的仓库中

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/mystore.png)

## Clone

进到自己 fork 的项目中，就能看到 `Code` 按钮，点击根据仓库克隆到本地

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/clonebutton.png)

**由于 antd 下载实在太慢，我改用 dayjs 为例**

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/downloaddayjs.png)

下载完成

## 修改远程仓库

接下来执行完毕，使用`git remote -v `查看当前状态，会出现这样的反馈信息

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/dayjsurl.png)

这个是我自己的项目地址

到项目的 github 中，复制项目的链接，继续查看当前状态

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/connecttimejs.png)

现在你的本地代码已经与远程代码相连了。

一定确定`origin`是你自己的地址，`upstream`是远程的地址。

## 代码本地操作

在本地创建分支，在分支上进行编辑代码，提交代码

````js
    git branch 新分支（develop）

    git checkout develop

    // 在本地编辑改变代码

    git add .

    git status

    git commit -m '提交本地代码'

    git push origin master
    ```
````

## 提交 PR

这时候，我们的代码已经被提交上来了，我们现在可以点击 pull request

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/pullcode.png)

然后，写上自己的说明提交即可

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/pullrequest.png)
