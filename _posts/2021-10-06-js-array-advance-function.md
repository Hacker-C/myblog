---
layout: post
title: "JavaScript 数组高阶方法"
subtitle: "JavaScript 数组的力量隐藏在数组方法中，而这其中最核心的是数组高阶方法。"
date: 2021-10-06
author: "MurphyChen"
header-img: https://cdn.jsdelivr.net/gh/Hacker-C/Picture-Bed@main/FrontEnd/js1.192lrbbkjc74.png
tags:
  - JavaScript
---

## 1. 什么是高阶方法

数组的方法其实是一种函数，所以数组的高阶方法实际上就是高阶函数。高阶函数是指将 **函数作为参数** 或者 **返回值为函数** 的一类函数。  

这些数组高阶方法有：`forEach()`、 `map()`、`filter()`、`some()`、`every()`、`reduce()` 等等。这些方法都接收一个回调函数作为参数，来对数组中元素进行相关操作。这个回调函数可以是 `function` 匿名函数，也可以是 `=>` 箭头函数。

## 2. 高阶方法的使用

### 2.1 forEach

`forEach` 方法用于遍历数组，不对原数组进行修改，无返回值。

```js
array.forEach(function(currentValue, index, arr));
```

该方法接收一个函数 `function` 参数，其中，该函数内又有三个参数：
- `currentValue`：数组当前项的值
- `index`：可选。数组当前项的索引
- `arr`：可选。数组对象本身
- `thisArg`：可选。当执行回调函数 `callback` 时，用作 `this` 的值。

一般可以省略后面回调函数中后两个参数和 `thisArg` 参数，即：
```js
array.forEach(function(currentValue));
```

Eg.1：遍历数组

```js
let colors = ['red', 'green', 'blue'];
colors.forEach(v => {
    console.log(v);
})
/* red green blue */
```

Eg.2：强制 `return` 无法提前停止该方法的执行。  
对于一般的函数，我们可以通过满足一定条件然后使用 `return` 提前退出方法，但这不适用于 `forEach` 方法。

```js
let arr = [1, 2, 3, 4, 5];
arr.forEach(v => {
    console.log(v);
    if (v > 3) return;
})
/* 1 2 3 4 5 */
```

### 2.2 map

`map()` 方法遍历一个数组，首先创建一个新数组，新数组中元素是旧数组中的元素调用一次所提供的函数参数后的返回值，最后 **返回这个新数组**。

```js
let newArray = array.map(function (currentValue, index, arr));
```

`map` 方法接收一个 `function` 参数，该函数有三个参数：
- `currentValue`：数组当前项的值
- `index`：数组当前项的索引
- `arr`：方法调用的数组对象本身

Eg1:将数组中每个元素进行平方计算
```js
let arr = [1, 2, 3, 4, 5];
// 配合箭头函数可以很简洁的实现
let ans = arr.map(x => x ** 2);
console.log(ans);
// [ 1, 4, 9, 16, 25 ]
```

### 2.3 filter

```js
array.filter(function (currentValue, index, arr));
```

`filter()` 方法创建一个新的数组，新数组中的元素是通过检查指定数组中符合条件的所有元素，主要 **用于筛选数组**，注意它 **返回一个新数组**。
- `currentValue`: 数组当前项的值
- `index`：数组当前项的索引
- `arr`：数组对象本身

Eg1:筛选出数组中的奇数
```js
let nums = [1, 0, 2, 3, 4, 5, 11, 20];
let ans = nums.filter(v => v % 2 === 1);
console.log(ans);
// [ 1, 3, 5, 11 ]
```

### 2.4 reduce

`reduce` 方法接收一个 `reducer` 函数作为 “累加器”，对数组中每一个元素执行该函数，最后 “累加” 为一个值作为返回结果。

```js
arr.reduce(reducer (accumulator, currentValue, index, array), initialValue]);
```
- `reducer`：累加器，可以是匿名函数。
- `accumulator`：累计器累计回调的返回值，它是 **上一次调用回调时返回的累积值** 或 initialValue（见下）。
- `currentValue`：遍历数组中的当前元素。
- `index`：当前数组元素的索引。
- `array`：原数组。
- `initialValue`：作为第一次调用 reducer 函数时的第一个参数 accumulator 的值。 如果没有提供初始值，则将使用数组中的第一个元素。

Eg1：数组求和  
分析：`pre` 时上一轮执行回调函数的返回值，`cur` 是当前元素。每次将上一轮结果与 `cur` 相加然后返回，就达到了求和的目的。

```js
let arr = [1, 2, 3, 4, 5];
let ans = arr.reduce((pre, cur) => pre + cur, 0);
// 15
```

Eg2：在没有初始值 `initialValue` 的空数组上使用 `reduce` 方法将报错。  
由于没有提供 `initialValue` 参数，于是第一次回调时，函数试图找原数组第一个元素作为 `initialValue` 的值。但由于数组为空，所以找不到初始值，报错。
```js
let arr = [];
let ans = arr.reduce((pre, cur) => pre + cur);
// 运行报错
```
> 所以以后使用 `reduce` 对数组求和，最好都要给 `initialValue` 初始值。

Eg3：统计字符串中每个字符出现的次数
```js
let s = ['a', 'a', 'b', 'c', 'c', 'c'];
let res = s.reduce((preObj, curValue) => {
    if (preObj[curValue]) {
        preObj[curValue]++;
    } else {
        preObj[curValue] = 1;
    }
    return preObj;
}, {});
console.log(res);
// { a: 2, b: 1, c: 3 }
```

Eg4：数组去重

```js
let arr = [1, 2, 3, 4, 4, 4, 5, 5, 6];
let res = arr.reduce((preArr, curValue) => {
    if (preArr.indexOf(curValue) === -1) {
        preArr.push(curValue);
    }
    return preArr;
}, []);
console.log(res);
// [ 1, 2, 3, 4, 5, 6 ]
```

### 2.5 some

```js
array.some(function(currentValue, index, arr));
```

`some()` 方法用于检测数组中 **是否至少有一个元素满足指定条件**。注意它返回值是布尔值，如果找到这个元素，就返回 `true`，如果找不到就返回 `false`。找到则直接返回。

- `currentValue`: 数组当前项的值
- `index`：数组当前项的索引
- `arr`：数组对象本身

Eg1：只要至少有一个偶数就返回 `true`
```js
let arr = [1, 3, 5, 6, 7, 9];
let res = arr.some(v => v % 2 === 0);
console.log(res); // true
```

### 2.6 every

```js
array.every(function(currentValue, index, arr));
```

`every()` 方法用于检测数组中的元素 **是否全部都满足指定条件**，若全满足则返回 `true`，否则返回 `false`（只要有一个不满足就返回 `false`）。
注意 **它返回值是布尔值**，如果查找到这个元素，就返回 `true`，如果查找不到就返回 `false`。

- `currentValue`: 数组当前项的值
- `index`：数组当前项的索引
- `arr`：数组对象本身

Eg1：数组中全部都是奇数才返回 `true`
```js
let nums = [1, 3, 5, 7, 9, 11];
let res = nums.every(v => v % 2 === 1);
console.log(res); // true
```

> 本文结束，感谢阅读。