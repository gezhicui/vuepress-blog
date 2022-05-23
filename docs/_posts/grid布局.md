---
title: grid布局
date: 2020-09-08 21:38:06
tags:
  - 前端
  - css
categories:
  - css
permalink: /pages/174442/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

## 什么是 grid 布局？

&emsp;&emsp;Flex 布局是轴线布局，只能指定"项目"针对轴线的位置，可以看作是`一维布局`，Grid 布局则是将容器划分成"行"和"列"，产生单元格，然后指定"项目所在"的单元格，可以看作是`二维布局`，Grid 布局远比 Flex 布局强大.但是兼容性没有 flex 布局好

## 基本概念

grid 布局中有两个基本概念：

- 容器

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/grid容器.png)

- 项目

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/grid项目.png)

- 行、列

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/grid基本概念.png)

## 容器属性

### grid-template-columns/rows

&emsp;&emsp;你想要多少行或者列，就填写相应属性值的个数，不填写，自动分配

```css
/* 指定列的数量和高度 */
grid-template-columns
/* 指定行的数量和高度 */
grid-template-rows
```

&emsp;&emsp;来写个代码感受下

```html
<div class="main">
  <div class="item item-1">1</div>
  <div class="item item-2">2</div>
  <div class="item item-3">3</div>
  <div class="item item-4">4</div>
  <div class="item item-5">5</div>
  <div class="item item-6">6</div>
  <div class="item item-7">7</div>
  <div class="item item-8">8</div>
  <div class="item item-9">9</div>
  <div class="item item-10">10</div>
</div>
```

```css
.main {
  display: grid;
  width: 600px;
  height: 600px;
  border: 10px solid skyblue;
  grid-template-columns: 100px 100px 100px;
}
```

然后再给每个 div 加上颜色，就有这样的效果：

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/gridcolunms.png)

如果我们想要让行元素也 100px，那么就只要加上`grid-template-rows:100px 100px 100px 100px`.
![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/grid实例.png)

#### repeat()

我们一直写很多 px 很麻烦，可以用`repeat()`来简写

repeat():第一个参数是`重复的次数`，第二个参数是所要`重复的值`,所以，上面 grid 布局的 css 可以写成：

```css
.main {
  display: grid;
  width: 600px;
  height: 600px;
  border: 10px solid skyblue;
  grid-template-columns: repeat(3, 100px);
  grid-template-rows: repeat(4, 100px);
}
```

#### auto-fill

有时，单元格的大小是固定的，但是容器的大小不确定，这个属性就会自动填充。就是说多出来的空间下一行会自己往后面填

```css
.main{
    display:grid;
    border:10px solid skyblue;
    grid-template-columns: repeat(auto-fill,100px）;
}

```

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/autofill.png)

#### fr

为了方便表示`比例关系`，网格布局提供了 fr 关键字（fraction 的缩写，意为"片段")

```css
.main {
  display: grid;
  width: 600px;
  height: 600px;
  border: 10px solid skyblue;
  /* 宽度平均分成3份 */
  grid-template-columns: repeat(3，1fr);
}
```

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/gridfr.png)

我们也可以不进行等分:

```css
.main {
  display: grid;
  width: 600px;
  height: 600px;
  border: 10px solid skyblue;
  /* 宽度平均分成3份 */
  grid-template-columns: 1fr 2fr 3fr;
  grid-template-rows: 100px 100px 100px 100px;
}
```

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/fr不等分.png)

#### minmax()

函数产生一个长度范围，表示长度就在这个范围之中，它接受两个参数，分别为最小值和最大值。

```css
.main {
  display: grid;
  border: 10px solid skyblue;
  /* 容器伸缩时左边的box为1fr，右边最大为1fr，最小为150px */
  grid-template-columns: 1fr minmax(150px, 1fr);
  grid-template-rows: 100px 100px 100px 100px;
}
```

#### auto

```css
.main {
  display: grid;
  border: 10px solid skyblue;
  /* 第一列100，第二列自适应，第三列100 */
  grid-template-columns: 100px auto 100px;
  grid-template-rows: 100px 100px 100px 100px;
}
```

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/gridauto.png)

### grid-row-gap/grid-column-gap

&emsp;&emsp;一话解释就是，item(项目）相互之间的距离

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/gap.png)

注意:根据最新标准，上面三个属性名的**grid-前缀已经删除**,` grid-column-gap`和`grid-row-gap`写成`column-gap`和`row-gap`，`grid-gap`写成`gap`

### grid-template-areas

一个区域由单个或多个单元格组成，由你决定(具体使用，需要在项目属性里面设置)

比如说，借用上一小节的图，我们可以写成这样：

```css
.main {
  /* 九个字母代表九个小格 */
  grid-template-areas: 'a b c' 'd e f' 'g h i';
  /* 竖着写，123小格属于同一个区域 */
  grid-template-areas:
    'a a a'
    'b b b'
    'c c c';
  /* 区域不需要利用，则使用"点”(.)表示 */
  grid-template-areas:
    'a . c'
    'b . f'
    'g . i';
}
```

&emsp;&emsp;区域的命名会影响到网格线。每个区域的起始网格线，会自动命名为区域名`-start`，终止网格线自动命名为区域名`-end`

### grid-auto-flow

&emsp;&emsp;划分网格以后，容器的子元素会按照顺序，自动放置在每一个网格。默认的放置顺序是"先行后列"，即先填满第一行，再开始放入第二行(就是子元素的主轴方向)

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/grid主轴.png)

\*在 grid-auto-flow 中，还有一个属性叫`dense`,能帮我们自动填充剩余的空间

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/dense.png)

### justfy-items/align-items

&emsp;&emsp;justfy-items(水平方向)/align-items(垂直方向)设置单元格内容的水平和垂直对齐方式

```css
justfy-items: start(靠左) | end(靠右) | center(居中) | streth(铺满);
align-items: start(靠左) | end(靠右) | center(居中) | streth(铺满);
```

&emsp;&emsp;`place-items`是`justfy-items`和`align-items`的简写,如下所示：

```css
/* justfy-items:center
align-items:center */
place-items: center center;
```

### justfy-content/align-content

&emsp;&emsp;justify-content(水平方向)/ align-content(垂直方向)设置`整个内容区域`的对齐方式

```css
justify-content: start | end | center | stretch | space-around I space-between |
  space-evenly;
align-content: start | end | center | stretch | space-around | space-between |
  space-evenly;
```

### grid-auto-columns/grid-auto-rows

&emsp;&emsp;用来设置`多出来的`items 宽和高

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/容器多出来的宽和高.png)

## 项目属性

### grid-column-start/grid-column-end/grid-row-start / grid-row-end

一句话解释，根据在哪根网格线用来指定 item 的具体位置.

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/网格线.png)

上图中有两行属性，分别是`grid-column-start`和`grid-column-end`,但是这两个属性可以简写:

```css
/* 1/3表示从第一根网格线开始，第三根网格线结束，并不是三分之一 */
grid-column: 1 / 3;
```

rows 也同理：

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/grid网格改.png)

#### span

我们还可以用 span 属性，span 属性代表`跨越几个网格`
以上例子中的代码等价于

```css
grid-colunm-strat:span 2
/* 等价于
grid-column-start:1;
grid-column-end:3; */

grid-colunm-end:span 2
/* 和grid-colunm-strat:span 2 等价 */
```

### grid-area

刚刚我们在容器属性中定义了`grid-template-areas`，现在我们可以定义`grid-area`来表示 item 在`grid-template-areas`中占了哪个区域,像这样：

```css
.main {
  grid-template-areas:
    'a a a'
    'b b b'
    'c c c';
}
.item {
  grid-area: b;
}
```

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/griditem.png)

### justify-self / align-self / place-self

&ensp;&emsp;`justify-self`属性设置单元格内容的水平位置（左中右)，跟`justify-items`属性的用法完全一致，但只作用于**单个项目(水平方向)**,也就是说如果全局设置了`justify-items`，但是其中有某个项目想用一个其他的样式，就可以使用`justify-self`

&ensp;&emsp;`align-self`属性设置单元格内容的垂直位置（上中下），跟`align-items`属性的用法完全一致，也是只作用于**单个项目(垂直方向)**

```css
justify-self: start | end | center | stretch;
```
