---
title: JS 函数式编程『简明教程』
date: 2022-04-07 15:01:06
tags:
  - JavaScript
layout: post
subtitle: '函数式编程——JS重要的编程思想'
author: 'MurphyChen'
header-img: /img/js.png
---

本文是一个 **简明** 的函数式编程入门，通过实际应用场景举例，带你快速学会 JS 函数式编程。涉及以下知识点：

1. 编程范式
2. 纯函数
3. 柯里化
4. 偏函数
5. 函数合成
6. 尾调用

## 1. 编程范式

编程范式，就是在软件开发过程中程序代码的 **典型的编程风格**。编程范式决定了程序员对程序编写和执行的看法，也决定了我们编写代码的方式和风格。

编程但是主要有以下几种

- 命令式编程
  - 面向过程编程
  - 面向对象编程
- 声明式编程
- 函数式编程

其实我们已经比较熟悉命令式编程了。例如，C 语言是面向过程编程的语言，而 C++/Java/JS/Pyhton 等语言中的类和对象支持了面向对象编程，要做一件事，步骤都描述得很清楚，这属于 **命令式编程**。

对于 Vue/React 等前端框架，以及 SQL、HTML、CSS 等，我们称之为 **声明式编程**，要干什么事，我们只要描述定义该动作，具体的动作由框架底层实现。例如，我们要修改一个页面的节点属性，在原生 JS 或者 jQ 中，都是先获取节点，然后设置节点的相应属性，这是过程式的。在 vue 里面，我们只需要修改绑定在 `data` 中的数据即可。

## 2. 函数式编程

### 定义

函数式编程（Functional Programming），就是 **把编程中各种运算和过程都转换成函数运算**。从定义可以看出，函数式和声明式有共同点，但函数式编程 **把声明封装成更抽象的函数了**。与过程式编程相比，**函数式编程更加强调程序执行的结果**。

例如，求数组 `arr` 的和，过程式编程风格是这样：

```js
let sum = 0
for (let i = 0; i < arr.length; i++) {
  sum += arr[i]
}
```

而函数式编程是这样，不需要关注具体是怎么求和的，更关注结果。函数内部 `reduce` 方法也是函数式的体现。

```js
function getSum(arr) {
  return arr.reduce((pre, cur) => pre + cur, 0)
}
let sum = getSum(arr)
```

### 特点

- 函数是一等公民
- 函数是纯函数

函数是一等公民（First Class）。函数与其他数据类型一样，处于平等地位，可以赋值给其他变量，也可以作为参数，传入另一个函数，或者作为别的函数的返回值。JS 很好的支持了这点，**JS 中的闭包、高阶函数、尾调用** 等等都说明：在 JS 中，函数是一等公民。

函数是纯函数（Pure Function）。纯函数要满足以下两点：

- 传入相同的参数总是返回相同的结果
- 没有副作用，内部执行不影响函数外的变量

## 3. 纯函数

**纯函数是函数式编程的基础。**

### 定义

前面提到过，纯函数要满足两个条件：

- 函数的返回结果只依赖于他的参数
- 函数没有副作用（指没有产生外部可观察到变化，例如改变了外部变量，打印出了数据）

例如，以下函数不是纯函数。

1.

```js
let c = 1
function f(a, b) {
  return a + b + c
}
console.log(f(1, 2)) // 4
c = 2
console.log(f(1, 2)) // 4
```

内部的结果收到外部 `c` 的影响，`c` 一旦修改，相同参数，结果就不一样。

2.

```js
let b = 10
function f(a) {
  b = 20
  return a * 2
}
```

有副作用，影响了外部变量 b。

3.

```js
function g(x) {
  console.log('hello')
  return x + 1
}
```

有副作用，打印了 hello

### 为什么要使用纯函数？

- 提高代码可读性。纯函数使该部分成为了独立的一部分，函数的功能可以通过只看该部分函数体就能知道。
- 利于调试。不会对外部有副作用，在代码中只要调试不会产生副作用的代码。
- 提高开发效率。多编写纯函数的代码。例如 `map`、`reduce`、`filter` 等等。

## 4. 柯里化

柯里化（curry）是一种将使用多个参数的一个函数转换成一系列使用一个参数的函数的技术。

例如，一段求三数之和的代码，这是非柯里化的：

```js
function add(x, y, z) {
  return x + y + x
}
add(1, 2, 3) // 6
```

柯里化之后，函数的参数每次只能传一个：

```js
function add(x) {
  return function (y) {
    return function (z) {
      return x + y + z
    }
  }
}
add(1)(2)(3) // 6
```

由于 **闭包和作用域链** 的作用，外层的函数参数能一直保存在执行上下文中，直到最终结果返回，而内层函数能够访问到外层函数的参数。从而能够最终进行累加。

箭头函数可以更加简洁的表示：

```js
const add = (x) => (y) => (z) => x + y + z
```

有一种特殊的柯里化，也就是后面的偏函数，它能够这样调用 `add(1)(2, 3)`，也就是每次传入的参数不限于一个，而是可能一个或者多个。**面试中也常考这种，实际应用场景也是这种，** 见下节。

## 5. 偏函数

### 定义

偏函数（Partial Function）就是 **把一个函数的某些参数先固化，也就是设置默认值，返回一个新的函数，在新函数中继续接收剩余参数**，这样调用这个新函数会更简单。

是函数柯里化的一种特殊使用场景，是一种 **高级的柯里化**。

### 举例

```js
const add = (x) => (y, z) => x + y + z
add(1)(2, 3) // 6
```

### 应用场景

- 参数复用
- 提前确认
- 延迟执行

1、参数复用

现在你要实现一个正则检验函数，要求根据传入的正则表达式 `regExp` 和 `txt` 来判断 `txt` 是否满足该正则式。

你可能会这样实现：

```js
function check(regExp, txt) {
  return regExp.test(txt)
}
```

然后调用：

```js
// 检测数字
check(/^\d+$/, 123)
check(/^\d+$/, 456)
check(/^\d+$/, 789)
// 检测全是小写字母
check(/^[a-z]+$/, 'abc')
check(/^[a-z]+$/, 'ddd')
check(/^[a-z]+$/, 'mnt')
```

发现没，每次检测不同类型的字符串的正则参数都不一样，这样每次都要写一遍参数，多麻烦呀！

我们可以使用 **偏函数** 解决这个问题，**把重复参数先固化起来，然后再去接收剩余参数**，这样就实现了 **参数复用**。

```js
// 偏函数（特殊柯里化）复用参数
const currring_check = function (regExp) {
  return function (txt) {
    return regExp.test(txt)
  }
}

const checkNumber = currring_check(/^\d+$/)
const checkLower = currring_check(/^[a-z]+$/)
```

使用的时候，没有冗余参数：

```js
// 检测数字
checkNumber(123)
checkNumber(456)
checkNumber(789)
// 检测全是小写字母
checkLower('abc')
checkLower('ddd')
checkLower('mnt')
```

2、提前确认

举例说明：对于事件监听方法（`addEventListener`、`attachEvent`）的兼容，**提前确定了会走哪一个方法，避免之后每次绑定事件都进行判断**。

```js
const whichEvent = (function () {
  if (window.addEventListener) {
    return function (element, type, listener, useCapture) {
      element.addEventListener(
        type,
        function (e) {
          listener.call(element, e)
        },
        useCapture
      )
    }
  } else if (window.attachEvent) {
    return function (element, type, handler) {
      element.attachEvent('on' + type, function (e) {
        handler.call(element, e)
      })
    }
  }
})()
```

使用方法：

```js
let p = document.querySelector('p')
whichEvent(p, 'click', function () {
  alert('click p')
})
```

3、延迟执行

延迟执行，就比如 `add(1)(2)(3)` 系列的例子，先不执行函数，先存储结果，最后一步再执行函数。_见下列面试题。_

### 面试题

1、请实现以下函数

```js
add(1, 2)(3)(4, 5) // 15
add(1, 2, 3)(4, 5) // 15
add(1)(2)(3)(4)(5) // 15
```

首先观察，这种函数调用的特点是每个 `add` 传入的 **总参数个数都相同（后面第 2 题属于参数个数不同的情况）**，则可以通过判断传入的参数个数是否达到了本来需要的参数（这里是 `5` 个），没达到个数就继续调用，用一个 `args` 存储后续来的参数；一旦达到个数，则执行使用 `call` 方法执行原始函数。

```js
// 原始函数
function sum(x, y, z, m, n) {
  return x + y + z + m + n
}

// 柯里化辅助函数
function curring(fn) {
  // 这种方法相当于把后续调用的参数通过 args 都收集起来
  return function _fn(...args) {
    // 当前参数总数达=原来函数参数数量时，则执行函数并返回结果
    // fn.length 是函数传入参数的个数
    if (args.length === fn.length) {
      return fn.call(this, ...args)
    }

    // 还没调用完，继续把后面调用的函数的参数添加到新的返回的函数中
    return (...remain) => {
      return _fn.call(this, ...args, ...remain)
    }
  }
}

// 柯里化后的函数
const add = curring(sum)
```

测试结果：

```js
add(1, 2, 3, 4, 5) // 15
add(1, 2)(3)(4)(5) // 15
add(1)(2, 3)(4, 5) // 15
```

2、请实现函数 `add`，要求实现以下调用结果：

```js
add(1, 2, 3, 4) + 1 // 11
add(1, 2)(3, 4) + 1 // 11
add(1)(2)(3) + 1 // 7
add(1, 2)(3) + 1 // 7
add(1)(2) + 1 // 4
```

思路分析：这种情况就属于 **每次传入参数总个数不同的情况**，而且这里有个 `+1` 的运算，说明前面的算式要进行 **隐式转换**。隐式转换会调用 `valueOf()` 或 `toString()` 方法，所以我们可以改写函数的 `toString()` 方法，把传入的参数一一累加，最后返回该函数。

`f.toString` 内部能够访问外部的 `args` 参数，形成了一个闭包，这个闭包将保存以前累加的参数。

这样，最后返回的函数的 `toString` 方法就包含了最终的结果，这个结果需要通过 **触发隐式转换** 得出。这也是为什么要 `+1`，**对象与原始类型相加会触发隐式转换**，也就触发了 `toString()`。

```js
function add(...args) {
  // bind返回新函数以满足偏函数保留参数
  var f = add.bind(null, ...args)

  // 将传入参数累加求和
  f.toString = () => {
    return args.reduce((a, b) => a + b, 0)
  }

  return f // 返回结果是一个函数，但 toString 结果为数字
}
```

## 6. 函数合成

### 定义

函数合成（compose）是指 **将多个函数合成一个函数**。

函数合成（函数组合）是函数式编程中基本的运算，和函数柯里化采用了相反的思想。JS 函数合成是 **把多个单一参数函数合成一个多参数函数的运算**。

### 举例

1. 一个很简单的函数合成，功能是先把传入的两个数先求和，然后翻倍，然后加一。

```js
const f = (x) => x + 1
const g = (x) => x * 2
const t = (x, y) => x + y
// 定义合成函数
const compose = (f, g, t) => (x, y) => f(g(t(x, y)))
// 合成
const fgt = compose(f, g, t)
console.log(fgt(1, 2)) // 7
```

2. 完成函数组合：先去除数组中的偶数，然后把每个元素求平方，然后再排序。

```js
// 完成函数组合：
// 先去除数组中的偶数，然后把每个元素求平方，然后返回逆序数组。
const t = (arr) => arr.reverse()
const g = (arr) => arr.map((x) => x ** 2)
const f = (arr) => arr.filter((x) => x % 2 !== 0)

const compose = (f, g, t) => (arr) => f(g(t(arr)))

const fgt = compose(f, g, t)

console.log(fgt([1, 2, 3, 4, 5])) // [ 25, 9, 1 ]
```

这里举的例子都是简单例子，实际上，每个函数的功能可能很复杂，那么这样分别定义不同功能的函数，然后组合。**在 debug 的时候，很清楚可以看出是哪个子函数出了问题，因为每个子函数都是纯函数，互不影响**。

## 7. 尾调用

## 8. 高阶函数

> 待更新...orz
