---
title: js中字符串常用方法
date: 2020-06-16 20:09:52
tags:
  - JavaScript
  - 字符串
categories:
  - JavaScript
permalink: /pages/0520fe/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

## 1、toLowerCase()

把字符串转为小写，返回新的字符串。

```javascript
var str = 'Hello World';
var str1 = str.toLowerCase();
console.log(str); //Hello World
console.log(str1); //hello world
```

---

## 2、toUpperCase()

把字符串转为大写，返回新的字符串。

```javascript
var str = 'hello world';
var str1 = str.toUpperCase();
console.log(str); //hello world
console.log(str1); //HELLO WORLD
```

---

## 3、charAt()

返回指定下标位置的字符。如果 index 不在 0-str.length(不包含 str.length)之间，返回空字符串。

```javascript
var str = 'hello world';
var str1 = str.charAt(6);
console.log(str1); //6
```

---

## 4、charCodeAt()

返回指定下标位置的字符的 unicode 编码,这个返回值是 0 - 65535 之间的整数。

```javascript
var str = 'hello world';
var str1 = str.charCodeAt(1);
var str2 = str.charCodeAt(-2); //NaN
console.log(str1); //101
```

---

## 5、indexOf()

返回某个指定的子字符串在字符串中第一次出现的位置

```javascript
var str = 'Hello World';
var str1 = str.indexOf('o');
var str2 = str.indexOf('world');
var str3 = str.indexOf('o', str1 + 1); //注意：indexOf()方法对大小写敏感，如果子字符串没有找到，返回-1。第二个参数表示从哪个下标开始查找，没有写则默认从下标0开始查找。
console.log(str1); //4 默认只找第一个关键字位置，从下标0开始查找
console.log(str2); //-1 没有找到
console.log(str3); //7
```

---

## 6、lastIndexOf()

返回某个指定的子字符串在字符串中最后出现的位置。

```javascript
var str = 'Hello World';
var str1 = str.lastIndexOf('o');
var str2 = str.lastIndexOf('world');
var str3 = str.lastIndexOf('o', str1 - 1);
console.log(str1); //7
console.log(str2); //-1
console.log(str3); //4
//注意：lastIndexOf()方法对大小写敏感，如果子字符串没有找到，返回-1。第二个参数表示从哪个下标开始查找，没有写则默认从最后一个字符处开始查找。
```

---

## 7、slice()

返回字符串中提取的子字符串。

```javascript
var str = 'Hello World';
var str1 = str.slice(2); //如果只有一个参数，则提取开始下标到结尾处的所有字符串
var str2 = str.slice(2, 7); //两个参数，提取下标为2，到下标为7但不包含下标为7的字符串
var str3 = str.slice(-7, -2); //如果是负数，-1为字符串的最后一个字符。提取从下标-7开始到下标-2但不包含下标-2的字符串。前一个数要小于后一个数，否则返回空字符串

console.log(str1); //llo World
console.log(str2); //llo W
console.log(str3); //o Wor
```

---

## 8、substring()

提取字符串中介于两个指定下标之间的字符。

```javascript
var str = 'Hello World';
var str1 = str.substring(2);
var str2 = str.substring(2, 2);
var str3 = str.substring(2, 7);
console.log(str1); //llo World
console.log(str2); //如果两个参数相等，返回长度为0的空串
console.log(str3); //llo W
//注意：substring()用法与slice()一样，但不接受负值的参数。
```

---

## 9、substr()

返回从指定下标开始指定长度的的子字符串

```javascript
var str = 'Hello World';
var str1 = str.substr(1);
var str2 = str.substr(1, 3);
var str3 = str.substr(-3, 2);
console.log(str1); //ello World
console.log(str2); //ell
console.log(str3); //rl
//注意：如果没有指定length,返回从下标开始处结尾处的所有字符串。
```

---

## 10、split()

把字符串分割成字符串数组。

```javascript
var str = 'AA BB CC DD';
var string1 = '1:2:3:4:5';
var str1 = str.split(''); //如果把空字符串 ("")用作分割符，那么字符串的每个字符之间都会被分割
var str2 = str.split(' '); //以空格为分隔符
var str3 = str.split('', 4); //4指定返回数组的最大长度
var str4 = string1.split(':');
console.log(str1); // ["A", "A", " ", "B", "B", " ", "C", "C", " ", "D", "D"]
console.log(str2); //["AA" "BB" "CC" "DD"]
console.log(str3); //["A", "A", " ", "B"]
console.log(str4); // ["1", "2", "3", "4", "5"]
```

---

## 11、replace()

在字符串中用一些字符替换另一些字符，或替换一个与正则表达式匹配的子串。

```javascript
var str = 'hello WORLD';
var reg = /o/gi; //o为要替换的关键字，不能加引号，否则替换不生效，i忽略大小写，g表示全局查找。
var str1 = str.replace(reg, '**');
console.log(str1); //hell** W**RLD
```

---

## 12、match()

返回所有查找的关键字内容的数组。

```javascript
var str = 'To be or not to be';
var reg = /to/gi;
var str1 = str.match(reg);
console.log(str1); //["To", "to"]
console.log(str.match('Hello')); //null
```

原文地址：[JS 字符串常用方法总结](https://www.cnblogs.com/Yimi/p/10362214.html)
