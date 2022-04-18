---
title: add(1)(2)(3)你会写了，那么add[1][2][3]呢？
date: 2022-04-01 15:01:06
tags:
  - JavaScript
  - 面试题
layout: post
subtitle: '盘点那些偏门的 JS 面试题'
author: 'MurphyChen'
header-img: /img/js.png
---

首先来看一道面试题，你可以停下来，自己先思考一下这道题。

面试题：请定义 `add`，使得以下表达式可以成功计算结果。

```js
add[1][2][3] + 4 // 10
add[10][20] + 30 // 60
add[1000][2000][3000] + 4000 // 10000
```

乍一看这道题，有点像常考的柯里化，也就是 `add(1)(2)(3)` 这种，我相信大家已经都很熟悉函数的柯里化了。但是这道题不一样，那么应该如何处理呢？

下面我以我自己的思路过程，来带大家解析这道题。

## getter 方法？

分析表达式结构，可以看出 `a[1]` 这种形式有几种可能：从数组取值、从普通对象中取值，两者都涉及到 **取值**。看到取值我们可以想到 JS 的 `getter` 方法，所谓 `getter` 方法就是将一个对象属性绑定到查询该属性时将被调用的函数，即：一旦调用绑定的属性，就要调用的 `getter` 方法。

> `get`语法将对象属性绑定到查询该属性时将被调用的函数。—— [MDN getter](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/get)

那么是否可行呢？首先看看 `getter` 的使用：

```js
// 定义 obj 的一个属性 max，使得它总是返回 obj 的 arr 属性中的最大值
const obj = {
  arr: [3, 2, 4, 1],
  get max() {
    return Math.max.apply(null, this.arr)
  }
}
```

可以看出，`getter` 方法只能获取 **固定名字** 的属性（这里是 `max`），而我们需要的是不定参数名的 **数字** 。那么是不是可以通过 `getter` 方法传参，把数字当作参数传进去呢？不可行， `getter` 方法不能传参：

> get 语法必须不带参数。——[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/get#%E6%8F%8F%E8%BF%B0)

现在可以排除使用 `getter` 方法了。

## Proxy 代理

访问一个对象的属性，我们需要在 **访问** 这里下功夫，而 ES6 的 **poroxy 代理** 就是为此而生！

请看 MDN 对于 `Proxy` 的定义：

> Proxy  对象用于创建一个对象的代理，从而实现基本操作的拦截和自定义（如 **属性查找**、赋值、枚举、函数调用等）。

属性查找，没错，我们就是需要这个。我们将 `add[1]` 这种形式理解为获取一个属性，只不过这个属性是一个数字。对象的属性查找有两种方式，一类是 `obj.attr`（attr 必须是变量而非直接的字面量），另一种是 `obj[key]`（key 为变量或者字面量），这里的 `add[1]` 符合第二种属性查找方式。

> 这里我只是简单叙述一下 Proxy 的基本使用，详细请查阅 MDN 文档 或者 红宝书第九章。

举个例子，若 `proxy` 代理了 `obj`，那么 `proxy` 就成为了一个中间载体，通过 `proxy` 来间接访问 `obj`。为什么要增加一个中间载体，直接访问 `obj` 不行吗？这是因为我们可以设置 `proxy` 来进行进行一些拦截操作，比如非法访问的拦截，对返回结果进行一些处理。

`Proxy ` 代理的基本语法：

```js
const proxy = new Proxy(target, handler)
```

- `target`：代理的目标对象
- `handler`：一个以函数（方法）作为属性的对象，定义了在执行各种操作时代理所触发的行为。可以是一个 `get()` 方法，此方法定义了在获取/访问/查找目标对象的属性的时候，可以捕获这个行为，对其进行拦截或者进一步处理，又称为 `get` 捕获器。

`get` 捕获器的参数：

```js
const handler = {
  get(trapTarget, property, receiver) {
    // trapTarget：目标对象 target
    // property：要访问的属性，本题中都是一些数字
    // receiver：代理本身，也就是 proxy
  }
}
```

`Proxy ` 代理的举例使用：

```js
// 设置一个代理，每次访问 foo 属性，都返回 foo 的两倍
const target = { foo: 100 }
const handler = {
  get(trapTarget, property, receiver) {
    return 2 * trapTarget.foo
  }
}
const proxy = new Proxy(target, handler)
console.log(proxy.foo) // 200
```

好了，加入原来你不知道 Proxy，那么现在也大概能理解其基本使用了（至少对于本题而言足够看懂下面的题解）。

## 最终解决方案

有了以上分析和基础知识，现在可以来解决这道题了。（请结合注释和后面的关键解释理解）

```js
// 被代理的目标对象，有一个属性 sum，用来进行累加
const target = { sum: 0 }

// handler 参数，定义一个 get 捕获器
const handler = {
  get(trapTarget, property, receiver) {
    // 原表达式中的 + 操作会触发代理的 toPrimitive 隐式转换 ...(1)
    // 在代理对象中具体以 Symbol.toPrimitive 的形式存在
    if (property === Symbol.toPrimitive) {
      // 最后一次触发的是加法，则返回累加的结果
      // 这里需要暂时储存结果，因为要清空 0，以进行下一次表达式计算 ...(2)
      let temp = trapTarget.sum
      // 清空 sum 属性
      trapTarget.sum = 0
      // Symbol.toPrimitive 是一个对象内部的【函数属性】...(3)
      // 内部需要执行该函数，因此套了一个箭头函数，执行结果是返回累积和
      return () => temp
    }
    // 访问的属性为数字，会被转为字符串，因此要转回数字 ...(4)
    trapTarget.sum += Number(property)
    // 返回代理本身，以进行下一次访问，达到 add[1][2][3] 连续访问的目的
    return receiver
  }
}
const add = new Proxy(target, handler)
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b608a724bf87466e86970b6b1c4541fe~tplv-k3u1fbpfcp-watermark.image?)

关键点解释：

（1）对于 `add[1][2][3] + 4`，最后一步是 `+` 操作，熟悉隐式转换的同学肯定一眼看出要进行隐式转换（不熟悉的可以站内搜索）。而最后肯定是一个代理对象（代码 第(5)步中的`receiver`）和一个数字（`4`） 相加，那么对象会触发 `toPrimitive` 隐式转换，会尽量转换为一个原始类型，才能和 `4` 相加。

> `Symbol.toPrimitive`  是一个内置的 Symbol 值，它是作为对象的函数值属性存在的，当一个对象转换为对应的原始值时，会调用此函数。——[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/toPrimitive)

所以第 (1) 步中判断了 `property === Symbol.toPrimitive` ，若触发 `toPrimitive`，则要传入 `Symbol.toPrimitive` 作为访问属性，则说明进行了加法。那么就直接返回累加结果：` trapTarget.sum`。

（2）第 2 步之所以要清空 `sum` 属性，是因为我们代理的始终是同一个 `target` 对象，不同表达式之间的 `sum` 属性是共享的，不清空会影响下一个表达式的结果。

（3）`Symbol.toPrimitive` 是对象内部的一个 **函数属性**，需要在 `get` 中被执行（将对象转为原始类型），所以才需要返回一个函数：` return () => temp`，执行后返回累加结果 `sum`。

（4）对象访问数字属性的时候（指通过 `obj[key]`）访问，若 `key` 为数字，则会隐式转换为字符串。所以第 4 步需要 `Number(property)` 转换为数字后才能参与累加。  
不信你可以跑跑下面代码：

```js
const obj = { 100: 'foo' }
console.log(obj[100]) // foo
```

以上是本题的全部内容，有关 Proxy 的更深入理解和使用还请自行查看文档学习噢。
