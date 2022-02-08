---
title: JavaScript 删除对象属性
date: 2022-02-08 17:01:06
tags:
  - JavaScript
layout: post
subtitle: "在 JS 中如何删除一个对象 object 的属性？"
author: "MurphyChen"
header-img: https://cdn.jsdelivr.net/gh/Hacker-C/Picture-Bed@main/FrontEnd/js1.192lrbbkjc74.png
---

## 需求

有下面一个对象，现在有一个需求，要删除该对象的 `age` 属性。
```js
let obj = {
  id: '1001',
  name: 'peter',
  age: 18
}
```

## 1. 方法一：使用 delete

```js
let obj = {
  id: '1001',
  name: 'peter',
  age: 18
}
delete obj.age
console.log(obj)
// {"id":"1001","name":"peter"}
```

注意：  
如果对象的原型链上有一个与待删除属性同名的属性，那么删除属性之后，对象会使用原型链上的那个属性（也就是说，delete操作只会在自身的属性上起作用）  

```js
function Person() {
  this.name = 'murphy'
  this.age = 18
}
Person.prototype.age = 20
let f = new Person()
console.log(f.age) // 18
delete f.age
console.log(f.age) // 20
```

## 2. 方法二：对象解构

```js
let obj = {
  id: '1001',
  name: 'peter',
  age: 18
}

let { age, ...newObj } = obj
console.log(newObj) // { id: '1001', name: 'peter' }
```

注意：对象的解构与数组有一个重要的不同。数组的元素是按次序排列的，变量的取值由它的位置决定；而 **对象的属性没有次序，变量必须与属性同名，才能取到正确的值**。即等号左边的两个变量的次序，与等号右边两个同名属性的次序不一致，对取值完全没有影响。  
另外，这里的 `...newObj` 是 ES6 的 rest 参数的应用。

封装一个删除对象属性值的函数：
```js
// 封装函数
const removeProperty = (propKey, { [propKey]: v, ...rest }) => rest

let obj = {
  id: '1001',
  name: 'peter',
  age: 18
}
obj = removeProperty('age', obj)
console.log(obj) // { id: '1001', name: 'peter' }
```

## 3. 方法三：对象拷贝

```js
const removeProperty = (propKey, obj) => {
  let res = {}
  // 通过 filter 数组方法过滤掉待删除的属性，然后将剩下的属性拷贝到新对象
  Object.keys(obj)
    .filter(key => key != propKey)
    .forEach(item => {
      res[item] = obj[item]
    })
  return res
}

let obj = {
  id: '1001',
  name: 'peter',
  age: 18
}
obj = removeProperty('age', obj)
console.log(obj) // { id: '1001', name: 'peter' }
```
