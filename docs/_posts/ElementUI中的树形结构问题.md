---
title: ElementUI中的树形结构问题
date: 2020-12-23 15:36:04
tags:
  - Vue
  - ElementUI
categories:
  - 框架

permalink: /pages/3fcbb0/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

最近实习了，很久没写博客，今天来记录一下使用` ElementUI`的树行控件遇到的问题

## 问题一 ：elementui 中树形结构问题

### 问题起因

在` ElementUI`中，我们想要渲染一颗树结构，像这样：
![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/树形控件.png)

那么我们从后端拿到的数据必须是层层嵌套`children`属性的一个对象:

```js
data: [{
          label: '一级 1',
          children: [{
            label: '二级 1-1',
            children: [{
              label: '三级 1-1-1'
            }]
          }]
        }, {
          label: '一级 2',
          children: [{
            label: '二级 2-1',
            children: [{
              label: '三级 2-1-1'
            }]
          }, {
            label: '二级 2-2',
            children: [{
              label: '三级 2-2-1'
            }]
          }]
        }, {
          label: '一级 3',
          children: [{
            label: '二级 3-1',
            children: [{
              label: '三级 3-1-1'
            }]
          }, {
            label: '二级 3-2',
            children: [{
              label: '三级 3-2-1'
            }]
          }]
        }],
```

问题来了，后端很忙，没空改接口，所以他返回的数据是这样的：

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/接口传值.png)

自身的唯一 id 为`depCode`,与父节点关联的 id 为`parentCode`。这样就不是`elementUI`中定义的数据格式。那么我们就需要做转换，把这样一个数组转换成为`elementui`中所需要的数据格式

### 形成树形结构代码

```js
//传进来的值为后端接口获取到的数组
    toTree (data) {
      // data.forEach(function (item) {
      //   delete item.children
      // })
      var map = {}
      //把所有数据放进map中
      data.forEach(item => {
        map[item.depCode] = item
      })
      //定义一个空数组，存放我们需要的结构的数组
      var val = []
      //遍历所有数组项
      data.forEach(item => {
        //parent为该项中parentCode对应的项
        var parent = map[item.parentCode]
        //如果parent存在
        if (parent) {
          //判断parent是否已经有children，没有则创建该数组，并加入到其children中
          (parent.children || (parent.children = [])).push(item)
        } else {
          //parent不存在，item加入到顶层对象中
          val.push(item)
        }
      })
      console.log(val)
      return val
    }
```

大功告成，通过遍历，我们就可以得到一个符合规则的数组了。

## 问题二 ：elementui 中的级联选择器问题

### 问题起因

在`elementUI`的级联选择器中，我们需要一个所属分组，我做完的效果是这样：
![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/级联选择器完成样式.png)

但是会有一个问题，这个级联选择器绑定的数据类型是和树形数据一样的，子级都需要`children`属性。所以我把`toTree()`完的数组传了进来。每一个项都有其`depId`，在调用接口保存的时候我们把该处的`depId`保存就可以。但是级联选择器在选择的时候，会把父节点也一起选择了，像这样:
![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/二级级联选择器.png)

或者这样：

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/级联选择器三级.png)

这个问题到还简单，因为我们始终选择需要最后一级的数据，所以我们只需要向接口传

```js
this.parentArr[this.parentArr.length - 1];
```

就行了，但是在回显的时候，级联选择器需要这样的一个包含**所有父级项**的数组才能回显。这就需要我们用现在的`depId`逆推整棵树结构获取到所有其父级的`depId`。我写了一个深度搜索算法，如下：

### 算法实现

```js
// 深度搜索查找子节点的所有父节点,传入当前的**depId**和**整个树结构**
    findParentDepId (depId, treeData) {
      treeData.forEach(item => {
        //把第一个数据放入根节点
        this.parentArr.push(item.depId)
        //如果找到便停止
        if (item.depId === depId) {
          console.log(this.parentArr)
          return 1
        } else {
          //如果没找到，且该节点有子节点
          if (item.children) {
            //递归继续遍历找
            this.findParentDepId(depId, item.children)
          } else {
            //找不到，数组清空，不影响下次判断
            this.parentArr = []
          }
        }
      })
    }
```

这样，我们就能从整颗树结构中遍历找到其所有的祖先元素的`depId`

结束！

--- 更新
在之前的代码中，由于遍历到最底如果没有找到`children`，就会清空整个数组，导致只能找到最后一项，所有我更新了一下：

```js
function findParent(treeData, id) {
  // 创建一个空数组
  var temp = [];
  // 递归函数
  var forFun = function (arr, id) {
    for (var i = 0; i < arr.length; i++) {
      // 循环当前层级的数据，存到item中
      var item = arr[i];
      //如果找到item的值等于id
      if (item.value === id) {
        // 向temp头添加一个函数
        temp.unshift(item.value);
        // 继续寻找他的上级
        forFun(treeData, item.parentId);
        break;
      } else {
        // 没找到，更深一级查找
        if (item.children) {
          forFun(item.children, id);
        }
      }
    }
  };
  forFun(treeData, id);
  return temp;
}
```
