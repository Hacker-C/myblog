---
title: a==1 && a==2 && a==3 的三种解法
date: 2022-04-17 15:01:06
tags:
  - JavaScript
  - 面试题
layout: post
subtitle: '隐式转换题，多种解法'
author: 'MurphyChen'
header-img: /img/js.png
---

## 问题提出

相信大家都很熟悉这个面试题了。但是今天我们要使用三种方法来解这道题，分析问题的过程中也可以查漏补缺、复习基础。看看哪种方法你不会。

经典问题：当 `a` 为何值，可使 `a` 成功打印？

```js
// let a = ??? 可使 a 成功输出
if (a == 1 && a == 2 && a == 3) {
  console.log(a)
}
```

至少有三种方法解：

1. `a` 为对象（`object`），重写 `valueOf` / `toString` 方法
2. `a` 为数组（`array`），重写 `join` 方法
3. `a` 为代理（`proxy`），构造 `get` 捕获器

## 解法 1：隐式转换之重写 `valueOf()` / `toString()`

这种解法是最常见的写法，直接重写 `valueOf` 或者 `toString`。

经过分析，`a` 不可能是一个原始数据类型，那么 `a` 就只能是一个复杂数据类型了，常见的有对象、数组（实际上数组也是一种特殊的对象）。

> 双等号 `==` 比较规则：如果操作数之一是对象，另一个是数字或字符串，会尝试使用对象的 `valueOf()` 和 `toString()` 方法将对象转换为原始值。
> —— [MDN 文档 ==](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Equality#%E6%8F%8F%E8%BF%B0)

分析 `a == 1 && a == 2 && a == 3`，若 `a` 为一个对象，那么 `a` 与数字类型之间的比较就会触发 `valueOf()` 方法，若此方法返回还是返回一个对象，则继续调用 `toString()` 方法，所以我们重写 `valueOf()` 或者 `toString()` 方法都可行。

思路：构造一个对象，有一个属性 `i=1`，改写 `valueOf` 方法，每次都返回 `i`，并让 `i++`。

代码：

```js
let a = {
  i: 1,
  valueOf() {
    return this.i++
  }
}

// 测试
if (a == 1 && a == 2 && a == 3) {
  console.log(a)
}

// 输出：{i: 4, valueOf: ƒ(){...}}
```

## 解法 2：使用 `Object.defineProperty` 改写数组 `join` 方法

让我们复习一遍刚刚 MDN 上的知识：

> 双等号 `==` 比较规则：如果操作数之一是对象，另一个是数字或字符串，会尝试使用对象的 `valueOf()` 和 `toString()` 方法将对象转换为原始值。

这里的对象也可以是数组，因此，数组与数字的比较也会尝试调用 `valueOf()` 和 `toString()` 方法。

可以结合下下面这个例子体会：

```js
let arr = [1, 2, 3]
let str = '1,2,3'
arr === str // true
```

上面这个例子可以验证 MDN 的这个隐式转换知识点。实际上，数组上的 `toString` 方法和 `join` 方法存在一定的联系，具体表现为：

1. `Array.prototype.toString() === Array.prototype.join()`（对于任何数组，默认方法也成立，且方法无参数）
2. 数组在调用 `toString` 方法的时候， 若 `toString` 方法没有被重写，而 `join` 被重写了，则数组会去调用 `join` 方法。

例如，我们改写了数组的 `join` 方法，但是没有改写 `toString` 方法，那么调用 `toString` 就会执行 `join` 方法。

```js
let arr = [1, 2, 3]
arr.join = () => 'join 被重写了'
arr.toString() // join 被重写了
```

以上分析表明：对于数组而言，隐式转换调用 `toString()`，相当于执行了 `join()`，我们就可以改写 `join` 方法。对于数组我们无法直接在 `let a = [1, 2, 3]` 里面增加方法，所以考虑使用 `Object.defineProperty()` 方法，

> [MDN - Object.defineProperty](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)

语法：

```js
Object.defineProperty(obj, prop, descriptor)
```

参数：

- `obj`：要定义属性或方法的目标对象
- `prop`：要定义的属性或方法名
- `descriptor`：一个对象，包含 `value` 等属性，具体可查阅文档

思路分析：这里我们不再使用前一种思维模式，设置 `i` 然后自增，我们可以让数组的 `join` 方法赋值为 `shift` 方法，这样每次与数字比较，都执行 `join`方法，间接的执行了 `shift` 方法，移除数组首位元素。

代码：

```js
let a = Object.defineProperty([1, 2, 3], 'join', {
  get: function () {
    return () => this.shift()
  }
})

// 测试
if (a == 1 && a == 2 && a == 3) {
  console.log(a)
}

// 打印：[join: (...), length: 0, get join: ƒ ()]
```

## 解法 3：使用 `Proxy` 代理构造 `get` 捕获器

ES6 的新特性（对于现在也不新了） `Proxy`，能够代理目标对象的一些行为，例如在获取目标对象的一些属性和方法的时候进行拦截或者进一步处理后再返回结果。

不熟悉的同学请查阅文档：

> [MDN - Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)

这个思路和第一种改写 `valueOf` 方法然后每次递增 `i` 的方法思想是一样的，只是换成了 `proxy` 代理模式。

代码：

```js
let a = new Proxy(
  { v: 1 },
  {
    get(target, property, receiver) {
      // 隐式转换会调用 Symbol.toPrimitive，这是一个函数
      if (property === Symbol.toPrimitive) {
        // 函数属性，所以要返回一个函数，会被自动执行
        return () => target.v++
      }
    }
  }
)

// 测试
if (a == 1 && a == 2 && a == 3) {
  console.log(a)
}

// 打印：Proxy {v: 4, ...}
```

本文完。
