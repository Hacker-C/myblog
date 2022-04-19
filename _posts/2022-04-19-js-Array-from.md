---
title: Array.from() 的妙用
date: 2022-04-19 14:04:06
tags:
  - JavaScript
layout: post
subtitle: '被忽略的 JS 数组技巧。'
author: 'MurphyChen'
header-img: /img/js.png
---

## 前置知识

`Array.from()` 接收一个类数组，或者一个可迭代对象（包括数组、Map 等），然后创建一个新的数组，并返回该新数组。

`Array.from(arrayLike, fn, thisArg)` 方法一共三个参数：

- `arrayLike`：要转换成数组的类数组或可迭代对象
- `fn`：可选，类数组或可迭代对象中的每一项都将执行该函数
- `thisArg`：- 可选参数，执行回调函数  `mapFn`  时  `this`  对象。

## 需求 1：`range(n)` 函数

要求：构造一个 `range(n)` 函数，返回一个 `[0, n)` 范围内的整数数组。例如 `range(5)` => `[0, 1, 2, 3, 4]`。

我们可以利用 `Array.from` 的第二个参数，以及第一个参数。我们使 `arrayLike = {length: 5}`，然后使 `fn = (_, i) => i`。

> 这里用到了类数组的概念，类数组是指有 `length` 属性，而且有可访问的索引属性（`index`）。所以这里使用了 `{ length: n }` 来构造一个类数组，使用 `(_, i) => i` 来对其中的每一项进行处理，`i` 是从 `0` 开始的索引属性。

完整代码如下：

```js
const range = (n) => Array.from({ length: n }, (_, i) => i)

range(5) // [0, 1, 2, 3, 4]
```

## 需求 2：`range(a, b)` 函数

要求：实现一个 `range(a, b)` 函数，返回一个整数序列数组。例如 `range(5, 10)` => `[5, 6, 7, 8, 9]`。

分析：首先是确定数组长度，很容易得出长度为 `b - a`，因此令 `length` 为 `b - a`。其次确定起始的数值，索引 `i` 从 `0` 开始，所以只需要 `i + a` 即可。

代码：

```js
const range = (a, b) => Array.from({ length: b - a }, (_, i) => i + a)

range2(5, 10) // [ 5, 6, 7, 8, 9 ]
```

## 需求 3：`range(a, b, step)` 函数

要求：实现一个 `range(a, b, step)` 函数，产生一个序列数组，数组中的值都相差 `step`，且范围为 `[a, b)` 内的整数。例如，`range(1, 10, 2)` => `[1, 3, 5, 7, 9]`。

分析：起始位置的值为 `a`，因为每个元素相隔 `step`，而且不能超过 `b`，因此元素的个数为 `(b - a) / step + 1`，然后将其向下取整，实际上 `Array.from({length: 5.8})` 会将 `length` 自动取整。逐渐递增 `step`，则传入的第二个参数为 `(_, i) => a + i * step`。

```js
Array.from({ length: 5.8 }, (_, i) => i) // [0, 1, 2, 3, 4]
```

代码：

```js
const range = (a, b, step) => Array.from({ length: (b - a) / step + 1 }, (_, i) => a + i * step)

range(1, 10, 2) // [ 1, 3, 5, 7, 9 ]
```

## 综合

实际上，这三种需求可以合并为一种，就是声明一个这样的函数 `range(a, b, step)`：

- 传入 `1` 个参数 `range(n)`，则返回 `[0, 1, 2, ..., n-1]`
- 传入 `2` 个参数 `range(a, b)`，则返回 `[a, a+1, ..., b-1]`
- 传入 `3` 个参数 `range(a, b, step)`，则返回 `[a, a + step, ...]`

三种情况合为一种，很简单，只需要将 **需求 3** 的参数设置默认值，然后判断是否传入了相关的参数即可。

代码：

```js
const range = (a, b, step = 1) => {
  if (b === undefined) {
    b = a
    a = 0
  }
  b--
  return Array.from({ length: (b - a) / step + 1 }, (_, i) => a + i * step)
}

range(10) // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
range(5, 10) // [ 5, 6, 7, 8, 9 ]
range(1, 13, 2) // [(1, 3, 5, 7, 9, 11)]
```

本文完。
