---
title: js中的常用数组方法
date: 2020-06-18 22:58:19
tags:
  - JavaScript
categories:
  - JavaScript
permalink: /pages/854bd8/
sidebar: auto
author:
  name: 杨雨翔
  link: https://gitee.com/xiang0515
---

**JavaScript 中创建数组有两种方式**

- 使用 Array 构造函数：

```js
var arr1 = new Array(); //创建一个空数组
var arr2 = new Array(20); // 创建一个包含20项的数组
var arr3 = new Array(“lily”,“lucy”,“Tom”); // 创建一个包含3个字符串的数组
```

- 使用数组字面量表示法：

```js
var arr4 = []; //创建一个空数组
var arr5 = [20]; // 创建一个包含1项的数组
var arr6 = [“lily”,“lucy”,“Tom”]; // 创建一个包含3个字符串的数组
```

**数组的方法有数组原型方法，也有从 object 对象继承来的方法，这里我们只介绍数组的原型方法，数组原型方法主要有以下这些:**

## 1、join()

join(separator): 将数组的元素组起一个字符串，以 separator 为分隔符，省略的话则用默认用逗号为分隔符，该方法只接收一个参数：即分隔符。

```js
var arr = [1, 2, 3];
console.log(arr.join()); // 1,2,3
console.log(arr.join('-')); // 1-2-3
console.log(arr); // [1, 2, 3]（原数组不变）
```

---

## 2、push()和 pop()

- push(): 可以接收任意数量的参数，把它们逐个添加到数组末尾，并返回修改后数组的长度。
- pop()：数组末尾移除最后一项，减少数组的 length 值，然后返回移除的项。

```js
var arr = ['Lily', 'lucy', 'Tom'];
var count = arr.push('Jack', 'Sean');
console.log(count); // 5
console.log(arr); // ["Lily", "lucy", "Tom", "Jack", "Sean"]
var item = arr.pop();
console.log(item); // Sean
console.log(arr); // ["Lily", "lucy", "Tom", "Jack"]
```

---

## 3、shift() 和 unshift()

- shift()：删除原数组第一项，并返回删除元素的值；如果数组为空则返回 undefined 。
- unshift:将参数添加到原数组开头，并返回数组的长度 。
  这组方法和上面的 push()和 pop()方法正好对应，一个是操作数组的开头，一个是操作数组的结尾。

```js
var arr = ['Lily', 'lucy', 'Tom'];
var count = arr.unshift('Jack', 'Sean');
console.log(count); // 5
console.log(arr); //["Jack", "Sean", "Lily", "lucy", "Tom"]
var item = arr.shift();
console.log(item); // Jack
console.log(arr); // ["Sean", "Lily", "lucy", "Tom"]
```

## 4、sort()

sort()：按升序排列数组项——即最小的值位于最前面，最大的值排在最后面。
在排序时，sort()方法会调用每个数组项的 toString()转型方法，然后比较得到的字符串，以确定如何排序。即使数组中的每一项都是数值， sort()方法比较的也是字符串，因此会出现以下的这种情况：

```js
var arr1 = ['a', 'd', 'c', 'b'];
console.log(arr1.sort()); // ["a", "b", "c", "d"]
arr2 = [13, 24, 51, 3];
console.log(arr2.sort()); // [13, 24, 3, 51]
//[1,2,3,5]
console.log(arr2); // [13, 24, 3, 51](原数组被改变)
```

为了解决上述问题，sort()方法可以接收一个比较函数作为参数，以便我们指定哪个值位于哪个值的前面。比较函数接收两个参数，如果第一个参数应该位于第二个之前则返回一个负数，如果两个参数相等则返回 0，如果第一个参数应该位于第二个之后则返回一个正数。以下就是一个简单的比较函数：

```js
function compare(value1, value2) {
  if (value1 < value2) {
    return -1;
  } else if (value1 > value2) {
    return 1;
  } else {
    return 0;
  }
}
arr2 = [13, 24, 51, 3];
console.log(arr2.sort(compare)); // [3, 13, 24, 51]
```

如果需要通过比较函数产生降序排序的结果，只要交换比较函数返回的值即可：

```js
function compare(value1, value2) {
  if (value1 < value2) {
    return 1;
  } else if (value1 > value2) {
    return -1;
  } else {
    return 0;
  }
}
arr2 = [13, 24, 51, 3];
console.log(arr2.sort(compare)); // [51, 24, 13, 3]
```

如果我们现再想在 Vue 实战中使用到 sort 写法，我们可以这么写：

```js
var app=new Vue({
            el:'#app',
            data:{
                items:[20,23,18,65,32,19,54,56,41]
            }
            methods:{
                function sortNumber(a,b){
                    return a-b
                }
            }
            computed:{
                sortItems:function(){
                    return this.items.sort(sortNumber);
                }
            }
        })
```

---

## 5、reverse()

reverse()：反转数组项的顺序。

```js
var arr = [13, 24, 51, 3];
console.log(arr.reverse()); //[3, 51, 24, 13]
console.log(arr); //[3, 51, 24, 13](原数组改变)
```

## 6、concat()

concat() ：将参数添加到原数组中。这个方法会先创建当前数组一个副本，然后将接收到的参数添加到这个副本的末尾，最后返回新构建的数组。在没有给 concat()方法传递参数的情况下，它只是复制当前数组并返回副本。

```js
var arr = [1, 3, 5, 7];
var arrCopy = arr.concat(9, [11, 13]);
console.log(arrCopy); //[1, 3, 5, 7, 9, 11, 13]
console.log(arr); // [1, 3, 5, 7](原数组未被修改)
```

从上面测试结果可以发现：传入的不是数组，则直接把参数添加到数组后面，如果传入的是数组，则将数组中的各个项添加到数组中。但是如果传入的是一个二维数组呢？

```js
var arrCopy2 = arr.concat([9, [11, 13]]);
console.log(arrCopy2); //[1, 3, 5, 7, 9, Array[2]]
console.log(arrCopy2[5]); //[11, 13]
```

上述代码中，arrCopy2 数组的第五项是一个包含两项的数组，也就是说 concat 方法只能将传入数组中的每一项添加到数组中，如果传入数组中有些项是数组，那么也会把这一数组项当作一项添加到 arrCopy2 中。

---

## 7、slice()

- slice(start, end) 方法可提取字符串的某个部分，并以新的字符串返回被提取的部分。
- 使用 start（包含） 和 end（不包含） 参数来指定字符串提取的部分。
- 字符串中第一个字符位置为 0, 第二个字符位置为 1, 以此类推。

提示： 如果是负数，则该参数规定的是从字符串的尾部开始算起的位置。也就是说，-1 指字符串的最后一个字符，-2 指倒数第二个字符，以此类推。

```js
//提取所有字符串:
var str = 'Hello world!';
var n = str.slice(0);
console.log(str); //Hello world!

//从字符串的第3个位置提取字符串片段:
var str = 'Hello world!';
var n = str.slice(3);
console.log(str); //lo world!

//从字符串的第3个位置到第8个位置直接的字符串片段:
var str = 'Hello world!';
var n = str.slice(3, 8);
console.log(str); //lo wo

//只提取第1个字符:
var str = 'Hello world!';
var n = str.slice(0, 1);
console.log(str); //H

//提取最后一个字符:
var str = 'Hello world!';
var n = str.slice(-1);
console.log(str); //!
```

---

## 8、splice()

splice()：很强大的数组方法，它有很多种用法，可以实现删除、插入和替换。

- 删除：可以删除任意数量的项，只需指定 2 个参数：要删除的第一项的位置和要删除的项数。例如， splice(0,2)会删除数组中的前两项。
- 插入：可以向指定位置插入任意数量的项，只需提供 3 个参数：起始位置、 0（要删除的项数）和要插入的项。例如，splice(2,0,4,6)会从当前数组的位置 2 开始插入 4 和 6。
- 替换：可以向指定位置插入任意数量的项，且同时删除任意数量的项，只需指定 3 个参数：起始位置、要删除的项数和要插入的任意数量的项。插入的项数不必与删除的项数相等。例如，splice (2,1,4,6)会删除当前数组位置 2 的项，然后再从位置 2 开始插入 4 和 6。
  splice()方法始终都会返回一个数组，该数组中包含从原始数组中删除的项，如果没有删除任何项，则返回一个空数组。

```js
var arr = [1, 3, 5, 7, 9, 11];
var arrRemoved = arr.splice(0, 2);
console.log(arr); //[5, 7, 9, 11]
console.log(arrRemoved); //[1, 3]
var arrRemoved2 = arr.splice(2, 0, 4, 6);
console.log(arr); // [5, 7, 4, 6, 9, 11]
console.log(arrRemoved2); // []
var arrRemoved3 = arr.splice(1, 1, 2, 4);
console.log(arr); // [5, 2, 4, 4, 6, 9, 11]
console.log(arrRemoved3); //[7]
```

## 9、indexOf()和 lastIndexOf()

- indexOf()：接收两个参数：要查找的项和（可选的）表示查找起点位置的索引。从数组的开头（位置 0）开始向后查找。
- lastIndexOf：接收两个参数：要查找的项和（可选的）表示查找起点位置的索引。从数组的末尾开始向前查找。
  这两个方法都返回要查找的项在数组中的位置，或者在没找到的情况下返回-1。在比较第一个参数与数组中的每一项时，会使用全等操作符。

```js
var arr = [1, 3, 5, 7, 7, 5, 3, 1];
console.log(arr.indexOf(5)); //2
console.log(arr.lastIndexOf(5)); //5
console.log(arr.indexOf(5, 2)); //2
console.log(arr.lastIndexOf(5, 4)); //2
console.log(arr.indexOf('5')); //-1
```

---

## 10、forEach()

forEach()：对数组进行遍历循环，对数组中的每一项运行给定函数。这个方法没有返回值。参数都是 function 类型，默认有传参，参数分别为：value(每个元素)；index（该元素索引）；arry（该数组本身）。

```js
var arr = [1, 2, 3, 4, 5];
arr.forEach(function (x, index, a) {
  console.log(x + '|' + index + '|' + (a === arr));
});
// 输出为：
// 1|0|true
// 2|1|true
// 3|2|true
// 4|3|true
// 5|4|true
```

---

## 11、map()

map()：指“映射”，对数组中的每一项运行给定函数，返回每次函数调用的结果组成的数组。
下面代码利用 map 方法实现数组中每个数求平方。

```js
var arr = [1, 2, 3, 4, 5];
var arr2 = arr.map(function (item) {
  return item * item;
});
console.log(arr2); //[1, 4, 9, 16, 25]
```

---

## 12、filter()

filter()：“过滤”功能，数组中的每一项运行给定函数，返回满足过滤条件组成的数组。

```js
var arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
var arr2 = arr.filter(function (value, index) {
  return index % 3 === 0 || value >= 8;
});
console.log(arr2); //[1, 4, 7, 8, 9, 10]
```

---

## 13、every()

every()：判断数组中每一项都是否满足条件，只有所有项都满足条件，才会返回 true。

```js
var arr = [1, 2, 3, 4, 5];
var arr2 = arr.every(function (value) {
  return value < 10;
});
console.log(arr2); //true
var arr3 = arr.every(function (value) {
  return value < 3;
});
console.log(arr3); // false
```

---

## 14、some()

some()：判断数组中是否存在满足条件的项，只要有一项满足条件，就会返回 true。

```js
var arr = [1, 2, 3, 4, 5];
var arr2 = arr.some(function (value) {
  return value < 3;
});
console.log(arr2); //true
var arr3 = arr.some(function (value) {
  return value < 1;
});
console.log(arr3); // false
```

---

## 15、reduce()和 reduceRight()

- 这两个方法都会实现迭代数组的所有项，然后构建一个最终返回的值。reduce()方法从数组的第一项开始，逐个遍历到最后。而 reduceRight()则从数组的最后一项开始，向前遍历到第一项。
- 这两个方法都接收两个参数：一个在每一项上调用的函数和（可选的）作为归并基础的初始值。
- 传给 reduce()和 reduceRight()的函数接收 4 个参数：**前一个值**、**当前值**、**项的索引**和**数组对象**。这个函数返回的任何值都会作为第一个参数自动传给下一项。第一次迭代发生在数组的第二项上，因此第一个参数是数组的第一项，第二个参数就是数组的第二项。

下面代码用 reduce()实现数组求和，数组一开始**加了一个初始值 10**。

```js
var values = [1, 2, 3, 4, 5];
var sum = values.reduceRight(function (prev, cur, index, array) {
  return prev + cur;
}, 10);
console.log(sum); //25
```

## 16、fill()

---2021.4.15 补充---

ES6 为 Array 增加了`fill()`函数，使用制定的元素填充数组，其实就是用默认内容初始化数组。

该函数有三个参数。

> arr.fill(value, start, end)

- value：填充值。
- start：填充起始位置，可以省略。
- end：填充结束位置，可以省略，实际结束位置是 end-1。

### 1.采用默认值填初始化数组

```js
const arr1 = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11];
arr1.fill(7);
console.log(arr1);
//7,7,7,7,7,7,7,7,7,7,7
```

### 2.制定开始和结束位置填充

实际填充结束位置是前一位。

```js
const arr3 = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11];
arr3.fill(7, 2, 5);
console.log(arr3);
//1,2,7,7,7,6,7,8,9,10,11
```

### 3.结束位置省略。

从起始位置到最后。

```js
const arr4 = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11];
arr4.fill(7, 2);
console.log(arr4);

// 1,2,7,7,7,7,7,7,7,7,7
```
