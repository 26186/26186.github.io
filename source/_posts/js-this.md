title: JS中this的理解
author: 26186
tags:
  - JavaScript
  - 前端
categories:
  - 前端
date: 2019-07-26 14:52:00
---
### this指向
开篇给结论：在 ES5 中 this 永远指向的是最后调用它的那个对象。
先给出一个简单的例子：
``` javascript
var obj = {
  func () {
    console.log(this)
  }
}
var bar = obj.func
bar() // 打印出window
obj.func() // 打印出obj
```
最后两行打印出来的结果为什么完全不一样，其实函数调用的本质是 call。
#### 函数调用的本质
```
func.call(context)
```
`context`就是函数调用时的执行上下文（即所在作用域）。谁调用它谁就是它的`context`，如果 context 是 null 或者是 undefined，context 就是 window。
`bar()`等价于**bar.call(undefined)**，为了更好的理解我们先看成是**window.bar.call(window)**
`obj.func()`等价于**obj.func.call(obj)**
> 综上所述可以粗暴的理解成谁`.`的它谁就是它的 this。

### 如何改变this指向
#### _this = this
在一些回调地狱中，最直接的办法就是提前存储 this。
```javascript
var callHell = {
  func() {
    var _this = this
    var callback = function() {
      console.log(this) // 打印出window
      console.log(_this) // 打印出callHell
    }
    callback()
  }
}
callHell.func()
```
> 结合之前的结论谁`.`的它谁就是它的this。
> `callback()`等价于`window.callback.call(window)`，所以`console.log(this)`打印出window。
> `callHell.func()`等价于`callHell.func.call(callHell)`，所以`var _this = this`的值为callHell，打印出callHell。
如果对`window.callback.call(window)`不太理解，其实原因很简单就是js中的var变量提升，可以看下面等价代码：

```javascript
var callBack = function(_this) {
  console.log(this) // 打印出window
  console.log(_this) // 打印出callHell
}
var callHell = {
  func(callback) {
    var _this = this 
    callback(_this)
  }
}
callHell.func(callBack)
```
#### call、apply、bind
前面一直在说函数的调用本质就是`func.call(context)`。所以自然可以使用 call 来改变 this 指向。apply、bind 和 call 是类似的。这三个区别可以自己查的了解下。
改造开篇第一个例子：
``` javascript
var callObj = {}
var obj = {
  func () {
    console.log(this)
  }
}
var bar = obj.func
bar() // 打印出window
bar.call(obj) // 打印出obj
bar.apply(obj) // 打印出obj
bar.bind(obj)() // 打印出obj
obj.func() // 打印出obj
obj.func.call(callObj) // 打印出callObj
obj.func.apply(callObj) // 打印出callObj
obj.func.bind(callObj)() // 打印出callObj
```
#### 箭头函数
先上结论：箭头函数没有 this 绑定，通过查找作用域链来决定 this 指向，如果箭头函数被非箭头函数包含，则 this 指向最近一层调用非箭头函数的那个对象，否则 this 为 undefined，undefined 时指向 window。
> ES5 的坑终于在 ES6 中得到解救。
 
改造万恶的回调地狱：
```javascript
var callHell = {
  func() {
    var callback = () => {
      console.log(this) // 打印出callHell
    }
    callback()
  }
}
callHell.func()
```
改造开篇第一个例子成 this 为 undefined情况：
``` javascript
var obj = {
  func: () => {
    console.log(this)
  }
}
obj.func() // 打印出window
```
结束时结论说三遍：
> 在ES5中 this 永远指向最后调用它的那个对象；在ES6中 this 永远指向最近一层调用非箭头函数的那个对象，否则 this 为 undefined，undefined时指向 window。
> 在ES5中 this 永远指向最后调用它的那个对象；在ES6中 this 永远指向最近一层调用非箭头函数的那个对象，否则 this 为 undefined，undefined时指向 window。
> 在ES5中 this 永远指向最后调用它的那个对象；在ES6中 this 永远指向最近一层调用非箭头函数的那个对象，否则 this 为 undefined，undefined时指向 window。