---
title: 深入理解 JavaScript 声明提升
date: 2021-10-14 14:13:00
tags:
  - JavaScript
layout: post
subtitle: "一文带你深入理解 JS 中的声明提升！"
author: "MurphyChen"
header-img: https://cdn.jsdelivr.net/gh/Hacker-C/Picture-Bed@main/FrontEnd/js1.192lrbbkjc74.png
---

## 1. 前言

JS 引擎在在执行 JavaScript 前会有预编译，JS 预编译时会把 JS 中的 `var` 变量声明和 `function` 函数声明提前至当前作用域的最前面。

传统艺能，先来看一个题，看看自己是否完全掌握了声明提升：  
以下代码的正确输出是？（答案在最好，如果你还不是很清楚的话，建议继续阅读下去。）
```js
var foo = function () {
    console.log(100);
}
function foo() {
    console.log(200);
}
foo();
```

## 2. 声明提升

JS 引擎在解释运行代码前，会首先处理任何代码中包括变量和函数在内的所有声明，将其放到当前作用域的最前方，而剩下的赋值语句则停留在原地等待执行。声明从原本的位置被提前到了当前作用域的最前方，因此称为 **声明提升（Hoisting）**。

### 2.1 变量声明提升

```js
// 输出 undefined
console.log(a);
var a = 'hello';
```

根据变量提升，变量 `a` 通过 `var` 声明，此声明 `var a` 将被提前，而 `a = 'hello'` 则停留在原地。所以该代码相当于：

```js
var a;
// 输出 undefined
console.log(a);
a = 'hello';
```

需要注意，变量的声明会提升，而变量的初始化不会被提升。

```js
// 输出 undefined
console.log(a);
var a;
a = 'hello';
```

代码中的每个作用域都会进行提升操作。
```js
console.log(a); // undefined
var a = 100;
function fn() {
    console.log(b); // undefined
    var b = 200;
    function gn() {
        console.log(c); // undefined
        var c = 300;
    }
    gn();
}
fn();
```

以上代码输出被提升处理后相当于：
```js
var a;
console.log(a); // undefined
a = 100;
function fn() {
    var b;
    console.log(b); // undefined
    b = 200;
    function gn() {
        var c;
        console.log(c); // undefined
        c =  300;
    }
    gn();
}
fn();
```

### 2.2 函数声明提升

通过 `function` 声明函数也具有声明提升特性，该函数声明（包括函数体）会提前至当前作用域的最前面。
```js
// 输出 hello
foo();
function foo() {
    console.log('hello');
}
```

以上代码相当于：
```js
function foo() {
    console.log('hello');
}
// 输出 hello
foo();
```

需要注意， `var` 声明的函数表达式不具有函数声明提升特性。例如以下匿名函数表达式：
```js
// 输出 undefined
console.log(foo);
foo(); //TypeError: foo is not a function
var foo = function () {
    console.log('hello');
}
```

以上代码相当于：
```js
var foo;
console.log(foo); // 输出 undefined
foo(); // TypeError: foo is not a function
foo = function () {
    console.log('hello');
};
```

对于具名函数表达式，同样没有声明提升。

```js
console.log(foo); // undefined
foo(); // TypeError: foo is not a function
var foo = function bar() {
    console.log('hello');
}
```

注意：具名函数表达式的名字只能在函数内部使用，外部不能使用。

```js
var bar = 10;
var foo = function bar() {
    bar = 30; // (3)
    console.log(bar); // (1)
}
foo();
console.log(bar); // (2)
bar(); // (4)
```
 
以上代码中，(1) 处输出：
```
ƒ bar() {
    bar = 32;
    console.log(bar);
}
```
(2) 处输出：`10`，(3) 处代码无效，(4) 处报错。

> ES6 提供的 `let` 和 `const` 声明不具有声明提升特性。

## 3. 函数声明优先

在代码中存在同名变量和函数的声明，那么两者均会提升，最后函数声明会覆盖变量声明。
```js
// 输出：foo () {}
console.log(foo);
var foo = 100;
function foo () {}
```

以上代码相当于：
```js
function foo() {};
var foo; // 变量提升被函数声明提升覆盖，变量重复声明无效
console.log(foo); // foo() {}
foo = 100;
```

但是在输出语句 `console.log` 前如果有赋值语句，则 `foo` 为变量的值。
```js
var foo = 100;
function foo () {};
console.log(foo); // 100
```

注意，**变量的重复声明是无用的，但函数的重复声明会覆盖前面的声明**。
- 变量的重复声明无用
    ```js
    var foo = 100;
    var foo;
    console.log(foo); // 100
    ```
- 函数的重复声明会覆盖前面的声明
    ```js
    foo(); // 200
    function foo () {
        console.log(100);
    }
    function foo () {
        console.log(200);
    }
    ```
- 函数声明提升优先于变量声明提升
    ```js
    foo(); // 200
    var foo = 100;
    function foo() {
        console.log(200);
    }
    ```
    以上代码相当于：
    ```js
    function foo() {
        console.log(200);
    }
    var foo;
    foo(); // 200，函数声明提升优先于变量声明提升，变量声明无效
    foo = 100;
    ```

## 4. 一些案例

案例1：
```js
var bar = 100;
function foo () {
    console.log(bar); // (1)
    var bar = 200;
    console.log(bar); // (2)
}
```

(1) 处将输出 `undefined`，(2) 处输出 `200`。根据变量提升原则，以上代码相当于：
```js
var bar = 100;
function foo () {
    var bar;
    console.log(bar); // undefined
    bar = 200;
    console.log(bar); // 200
}
```

案例2：
```js
var foo = function () {
    console.log(100);
}
function foo() {
    console.log(200);
}
foo();
```
以上代码将输出 `100`，根据 **函数声明提升优先原则**，代码可转换为：
```js
function foo() {
    console.log(200);
}
var foo;
foo = function () {
    console.log(100);
} // 原函数被变量赋值覆盖，此时 foo 函数体为后面赋值的。
foo(); // 100
```

## 本文参考

- [命名函数表达式 NFE](https://javascript.info/function-object#named-function-expression)
- [MDN 变量提升 Hoisting](https://developer.mozilla.org/zh-CN/docs/Glossary/Hoisting)