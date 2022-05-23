---
title: moment.js使用方法总结
date: 2020-11-03 22:59:44
tags:
  - moment
categories:
  - 前端
permalink: /pages/593dbb/
sidebar: auto
author:
  name: 杨雨翔
  link: https://gitee.com/xiang0515
---

&emsp;&emsp;`Moment.js`是一个轻量级的 JavaScript 时间库，它方便了日常开发中对时间的操作，提高了开发效率。日常开发中，通常会对时间进行下面这几个操作：比如获取时间，设置时间，格式化时间，比较时间等等。下面就是我对 moment.js 使用过程中的整理，方便以后查阅。

## 引入 moment.js

### 安装

> npm install moment --save 或者 yarn add moment

### 引入

```js
// require 方式
var moment = require('moment');
// import 方式
import moment from 'moment';
```

浏览器方式引入

```html
<script src="moment.js"></script>
```

## 设定 moment 区域为中国

```js
// require 方式
require('moment/locale/zh-cn');
moment.locale('zh-cn');

// import 方式
import 'moment/locale/zh-cn';
moment.locale('zh-cn');
```

## 方法速查

```js
moment() //获取时间

moment().startOf('?')//获取本周、本月的第一天0时0分0秒

moment().endOf('?')//获取本周、本月的最后一天23时59分59秒

moment().dayInMonth()//获取当月总天数

moment().unix()//返回时间戳（number）

moment().get('?')
moment().?()//获取某一事件

moment().set('?',num)
moment().?(num)//设置某一时间

moment().add(num,'?')//时间+num

moment().subtract(num,'?')//时间-num

moment().format('xxxxxxx')//格式化时间

diff()//比较时间

toDate()//时间转换成Date对象

fromNow()//相对现在的时间

calendar()//日期时间
```

## 基本使用

### 获取时间

#### 获取当前时间

```js
moment();
```

#### 获取今天 0 时 0 分 0 秒

```js
moment().startOf('day');
```

#### 获取本周第一天(周日 Sunday)0 时 0 分 0 秒

```js
moment().startOf('week');
```

#### 获取本周周一 0 时 0 分 0 秒

```js
moment().startOf('isoWeek');
```

#### 获取当前月第一天 0 时 0 分 0 秒

```js
moment().startOf('month');
```

#### 获取今天 23 时 59 分 59 秒

```js
moment().endOf('day');
```

#### 获取本周最后一天(周六)23 时 59 分 59 秒

```js
moment().endOf('week');
```

#### 获取本周周日 23 时 59 分 59 秒

```js
moment().endOf('isoWeek');
```

#### 获取当前月最后一天 23 时 59 分 59 秒

```js
moment().endOf('month');
```

#### 获取当前月的总天数

```js
moment().daysInMonth();
```

#### 获取时间戳(以秒为单位)

```js
moment().format('X'); // 返回值为字符串类型
moment().unix(); // 返回值为数值型
```

#### 获取时间戳(以毫秒为单位)

```js
moment().format('x'); // 返回值为字符串类型
moment().valueOf(); // 返回值为数值型
```

#### 获取年份

```js
moment().year();
moment().get('year');
```

#### 获取月份

```js
moment().month(); // (0~11, 0: January, 11: December)
moment().get('month');
```

#### 获取一个月中的某一天

```js
moment().date();
moment().get('date');
```

#### 获取一个星期中的某一天

```js
moment().day(); // (0~6, 0: Sunday, 6: Saturday)
moment().weekday(); // (0~6, 0: Sunday, 6: Saturday)
moment().isoWeekday(); // (1~7, 1: Monday, 7: Sunday)
moment().get('day');
mment().get('weekday');
moment().get('isoWeekday');
```

#### 获取小时

```js
moment().hours();
moment().get('hours');
```

#### 获取分钟

```js
moment().minutes();
moment().get('minutes');
```

#### 获取秒数

```js
moment().seconds();
moment().get('seconds');
```

#### 获取当前的年月日时分秒

```js
moment().toArray(); // [years, months, date, hours, minutes, seconds, milliseconds]
```

### 设置时间

#### 设置年份

```js
moment().year(2019);
moment().set('year', 2019);
moment().set({ year: 2019 });
```

#### 设置月份

```js
moment().month(11); // (0~11, 0: January, 11: December)
moment().set('month', 11);
```

#### 设置某个月中的某一天

```js
moment().date(15);
moment().set('date', 15);
```

#### 设置某个星期中的某一天

```js
moment().weekday(0); // 设置日期为本周第一天（周日）
moment().isoWeekday(1); // 设置日期为本周周一
moment().set('weekday', 0);
moment().set('isoWeekday', 1);
```

#### 设置小时

```js
moment().hours(12);
moment().set('hours', 12);
```

#### 设置分钟

```js
moment().minutes(30);
moment().set('minutes', 30);
```

#### 设置秒数

```js
moment().seconds(30);
moment().set('seconds', 30);
```

#### 年份+1

```js
moment().add(1, 'years');
moment().add({ years: 1 });
```

#### 月份+1

```js
moment().add(1, 'months');
```

#### 日期+1

```js
moment().add(1, 'days');
```

#### 星期+1

```js
moment().add(1, 'weeks');
```

#### 小时+1

```js
moment().add(1, 'hours');
```

#### 分钟+1

```js
moment().add(1, 'minutes');
```

#### 秒数+1

```js
moment().add(1, 'seconds');
```

#### 年份-1

```js
moment().subtract(1, 'years');
moment().subtract({ years: 1 });
```

#### 月份-1

```js
moment().subtract(1, 'months');
```

#### 日期-1

```js
moment().subtract(1, 'days');
```

#### 星期-1

```js
moment().subtract(1, 'weeks');
```

#### 小时-1

```js
moment().subtract(1, 'hours');
```

#### 分钟-1

```js
moment().subtract(1, 'minutes');
```

#### 秒数-1

```js
moment().subtract(1, 'seconds');
```

### 格式化时间

![](https://yangblogimg.oss-cn-hangzhou.aliyuncs.com/blogImg/格式化时间.png)

#### 格式化年月日： 'xxxx 年 xx 月 xx 日'

```js
moment().format('YYYY年MM月DD日');
```

#### 格式化年月日： 'xxxx-xx-xx'

```js
moment().format('YYYY-MM-DD');
```

#### 格式化时分秒(24 小时制)： 'xx 时 xx 分 xx 秒'

```js
moment().format('HH时mm分ss秒');
```

#### 格式化时分秒(12 小时制)：'xx:xx:xx am/pm'

```js
moment().format('hh:mm:ss a');
```

#### 格式化时间戳(以毫秒为单位)

```js
moment().format('x'); // 返回值为字符串类型
```

### 比较时间

#### 获取两个日期之间的时间差

```js
let start_date = moment().subtract(1, 'weeks');
let end_date = moment();

end_date.diff(start_date); // 返回毫秒数

end_date.diff(start_date, 'months'); // 0
end_date.diff(start_date, 'weeks'); // 1
end_date.diff(start_date, 'days'); // 7
start_date.diff(end_date, 'days'); // -7
```

### 转化为 JavaScript 原生 Date 对象

```js
moment().toDate();
new Date(moment());
```

### 日期格式化输出实例

```js
moment().format('MMMM Do YYYY, h:mm:ss a'); // 五月 24日 2019, 7:47:43 晚上
moment().format('dddd'); // 星期五
moment().format('MMM Do YY'); // 5月 24日 19
moment().format('YYYY [escaped] YYYY'); // 2019 escaped 2019
moment().format(); // 2019-05-24T19:47:43+08:00
```

### 相对时间输出实例

```js
moment('20111031', 'YYYYMMDD').fromNow(); // 8 年前
moment('20120620', 'YYYYMMDD').fromNow(); // 7 年前
moment().startOf('day').fromNow(); // 20 小时前
moment().endOf('day').fromNow(); // 4 小时内
moment().startOf('hour').fromNow(); // 1 小时前
```

### 日历时间输出实例

```js
moment().subtract(10, 'days').calendar(); // 2019年5月14日
moment().subtract(6, 'days').calendar(); // 上周六晚上7点49
moment().subtract(3, 'days').calendar(); // 本周二晚上7点49
moment().subtract(1, 'days').calendar(); // 昨天晚上7点49分
moment().calendar(); // 今天晚上7点49分
moment().add(1, 'days').calendar(); // 明天晚上7点49分
moment().add(3, 'days').calendar(); // 下周一晚上7点49
moment().add(10, 'days').calendar(); // 2019年6月3日
```

### 多语言支持输出实例

```js
moment().format('L'); // 2019-05-24
moment().format('l'); // 2019-05-24
moment().format('LL'); // 2019年5月24日
moment().format('ll'); // 2019年5月24日
moment().format('LLL'); // 2019年5月24日晚上7点50分
moment().format('lll'); // 2019年5月24日晚上7点50分
moment().format('LLLL'); // 2019年5月24日星期五晚上7点50分
moment().format('llll'); // 2019年5月24日星期五晚上7点50分
```

### 将毫秒数转为时分秒(可用于时间戳转换)

**注意：毫秒转为其他单位时，达到你想要转的单位时，为 1，超过时不管，不足时为 0；**

```js
const msTime = 4800000; //80分钟

moment.duration(msTime).hours(); //转为小时，值为1
moment.duration(msTime).minutes(); //转为分钟，值为20
moment.duration(msTime).seconds(); //转为秒，值为0
```

转为其他单位：

```js
moment.duration(msTime, 'seconds'); //转为秒
moment.duration(msTime, 'minutes'); //转为分
moment.duration(msTime, 'hours'); //转为小时
moment.duration(msTime, 'days'); //转为天
moment.duration(msTime, 'weeks'); //转为周
moment.duration(msTime, 'months'); //转为月
moment.duration(msTime, 'years'); //转为年
```

### 判断一个日期是否在两个日期之间

#### 不包含这两个起始日期

```js
this.moment('2010-10-20').isBetween('2010-10-19', '2010-10-25'); // true
this.moment('2010-10-19').isBetween('2010-10-19', '2010-10-25'); // false
this.moment('2010-10-25').isBetween('2010-10-19', '2010-10-25'); // false
```

#### 自定义是否包含起始日期

这里我们需要多用到两个参数：

- 第三个参数，固定为 null；
- 第四个参数，字符串，() 表示不包含，[] 表示包含。右括号开始日期，右括号控制结束日期

```js
this.moment('2016-10-30').isBetween('2016-10-30', '2016-12-30', null, '()'); //false
this.moment('2016-10-30').isBetween('2016-10-30', '2016-12-30', null, '[)'); //true
this.moment('2016-10-30').isBetween('2016-01-01', '2016-10-30', null, '()'); //false
this.moment('2016-10-30').isBetween('2016-01-01', '2016-10-30', null, '(]'); //true
this.moment('2016-10-30').isBetween('2016-10-30', '2016-10-30', null, '[]'); //true
```
