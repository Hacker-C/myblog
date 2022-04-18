---
title: JS实现三种脑洞大开的奇葩排序算法
date: 2022-04-05 09:01:06
tags:
  - JavaScript
layout: post
subtitle: '奇怪而又没啥用的排序算法'
author: 'MurphyChen'
header-img: /img/js.png
---

这次我们要讲的不是正经的冒泡排序、快排、堆排序之类的，而是三种奇葩的算法：

1. 睡眠排序（SleepSort）
2. 猴子排序（MonkeySort）
3. 面条排序（NoodlesSort）

实际上这三种算法并没有什么用，但很有趣。我要用 JS 实现，这里也涉及一些其他的算法和 JS 的特性，所以就当 **锻炼思维能力和熟悉 JS 特性**了。

## 睡眠排序

（1）原理：所谓 “睡眠”，就是根据数组元素的个数 `n` ，构造 `n` 个延时时间不同的线程，延时时间取决于数组中数字的大小。数值越小的元素，其构建的线程越早 **醒来**，从而就属于较小的元素，放入结果集中。这样，随着线程一一苏醒，我们就得到了一个排好序的数组。

（2）JS 实现分析：我们知道，JS 是一门单线程语言（永远只有一个线程在负责 JS 代码的执行），所以无法构造其他线程（这里不讨论 WebWorker 了）。但是我们有延时器函数 `setTimeOut`，可以用来 **模拟一个睡眠的线程**。

对于每个数字 `n` 构建的延时器，延时时间都设置为 `n * 4 + 4`，**数字越小，越先进入消息队列**。即：

```js
setTimeOut(callback, n * 4 + 4)
```

这里涉及到 `setTimeOut` 的细节问题：

1. `setTimeOut` 的回调函数会被添加到宏队列中，等待主线程调用执行。而多个延时时间不同的 **延时器回调函数添加的顺序是按照时间从短到长添加的，且是等时间到期了，然后再进入消息队列**。这样我们就可以保证数值小的数字先被添加了。
2. `setTimeOut` 有个最短延时时间 `4ms`，所以需要 `n * 4 + 4`，`+4` 是为考虑了 `0`。

（3）代码实现：

```js
// 模拟线程构造
async function generateThread(arr) {
  let res = await new Promise((resolve, reject) => {
    let res = []
    for (let a of arr) {
      setTimeout(() => {
        res.push(a)
        // 事件都添加完毕，则返回结果
        if (res.length === arr.length) resolve(res)
      }, a * 4)
    }
  })
  return res
}

async function monkeySort(arr) {
  let ans = await generateThread(arr)
  console.log(ans)
}

monkeySort([23, 3, 7, 6, 1, 7, 2, 10, 17])
// [1,  2,  3,  6, 7, 7, 10, 17, 23]
```

## 猴子排序

（1）原理：这个排序算法更加奇葩，不断循环随机产生传入数字的随机序列，直到产生的序列是有序的就停止循环，输出结果。取名猴子排序，参考了下面这条定理：

> 让一只猴子在打字机上随机地按键，当按键时间达到无穷时，几乎必然能够打出任何给定的文字，比如莎士比亚的全套著作。——维基百科

（2）JS 代码实现分析：首先是随机数组的生成，也就是 `shuffle` 化（洗牌）。很多人谈到随机打乱数组可能第一时间想到 `arr.sort((a, b) => Math.random() - 0.5)`，但实际上，这种方法很不合理，如果面试时让你写一个数组打乱算法，你写这种方法是不能让面试官满意的。具体原因，下篇文章讲。

更合理的数组打乱算法是 **洗牌算法**，不断循环从前 `n-1` 个数字中随机选出一个 数字，和当前位置 `n` **交换**，然后让 `n` 递减，直到打乱所有数字。

然后就是不断随机打乱数组，不断检测数组是否有序，直到检测出有序的，就返回结果。

（3）代码实现：

```js
// 洗牌算法，打乱数组
function shuffle(arr) {
  let len = arr.length
  while (len--) {
    let random = (Math.random() * len) >>> 0 // 移位取整
    ;[arr[random], arr[len]] = [arr[len], arr[random]] // 交换元素
  }
}

// 猴子排序
function monkeySort(arr) {
  while (true) {
    shuffle(arr)
    let flag = true
    // 检测随机生成的数组序列是否有序，一旦有序，返回结果
    for (let i = 0; i < arr.length - 1; i++) {
      if (arr[i] > arr[i + 1]) {
        flag = false
        break
      }
    }
    if (flag) return
  }
}

const arr = [23, 3, 7, 6, 1, 7, 2, 10, 17]
monkeySort(arr)
console.log(arr) // [1,  2,  3,  6, 7, 7, 10, 17, 23]
```

## 面条排序

（1）这个排序也很有意思，想象这样一个场景，你手里握着一把还没煮的面条（每根面条长度取决于每个数字大小，相当于模拟了每个数字）。现在，**把面条竖起来，把面条的下端放在在平整的桌面上，直到下端都与桌面对齐**。这样，每根面条上端距离桌面的高度就不一了。现在**用另一只手从上至下降低，先触碰到的就是较长的面条（较大的数字）**。

（2）JS 代码实现（含详细注释）

```js
function noodlesSort(arr) {
  let noodles = [] // 模拟面条
  let hand = -Infinity // 模拟手的位置，初始值为数组中最大的数字
  for (let a of arr) {
    // 模拟一根面条，数组最后一位存储的是面条长度（数组长度）
    let noodle = [...Array(a - 1).fill(''), a]
    noodles.push(noodle)
    if (a > hand) hand = a
  }
  // 存储答案
  let ans = []
  // 模拟用手向下接触面条
  while (hand--) {
    for (let noodle of noodles) {
      if (typeof noodle[hand] === 'number') {
        ans.push(noodle[hand])
      }
    }
  }
  return ans
}

noodlesSort([23, 3, 7, 6, 1, 7, 2, 10, 17])
// [ 23, 17, 10, 7, 7, 6,  3,  2, 1]
```

哈哈哈，是不是很有趣~

当然，这些算法都有很大缺陷和不好的复杂度，就当锻炼思维了。
