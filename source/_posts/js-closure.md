title: js中闭包的理解
author: 26186
tags:
  - JavaScript
  - ''
  - 前端
categories:
  - 前端
date: 2019-08-12 15:35:00
---
### 什么是闭包
> MDN上闭包的定义是`函数和声明该函数的词法环境的组合`。

这句话怎么理解呢？我们可以看下面的列子。

```javascript
var outer = function(){
  var count = 0;
  var inner = function(){
    count++;
    return count;
  }
  return inner;
}

var closure = outer();
```
![upload successful](/images/pasted-7.png)
使用 devtools 调试可以发现，图中的`Scope`是内部函数`inner`运行时的作用域，`Closure`就是闭包，里面的局部变量就是的声明该函数时的词法环境。

> 闭包是这样工作的，无论何时声明新函数并将其赋值给变量，都要存储函数定义和闭包。闭包包含在函数创建时作用域中的所有变量，它类似于背包。函数定义附带一个小背包，它的包中存储了函数定义创建时作用域中的所有变量。
> 通过闭包，我们可以在其他的执行上下文中，访问到函数的内部变量。

### 闭包的应用场景
#### 模块话
在jQuery中大量的使用到了模块化，以减少全局变量使用，避免变量污染。
```javascript
var objEvent = objEvent || {};
(function(){ 
  var addEvent = function(){ 
    // some code
  };
  function removeEvent(){
    // some code
  }

  objEvent.addEvent = addEvent;
  objEvent.removeEvent = removeEvent;
})();
```
#### 柯里化
```javascript
// 普通函数
function add(x, y) {
  return x + y
}

// 柯里化函数
function curryingAdd(x) {
  return function (y) {
    return x + y
  }
}
add(1, 1)
curryingAdd(1)(1)
```
#### 回调函数
现实开发中大量的回调函数，不再举例。
除了上面几个例子，我们在做 javacsript 开发时，随处可见闭包。那闭包又带来了哪些问题呢？
### 闭包的常见问题
#### 内存泄漏
闭包用的不好很容易导致内存泄露。
```javascript
function showId() {
  var el = document.getElementById("id")
  el.onclick = function(){
    // some code
  }
}
```
这样会导致闭包引用外层的el，当执行完showId后，el无法释放。
改成如下代码
```javascript
function showId() {
  var el = document.getElementById("id")
  var id  = el.id
  el.onclick = function(){
    // some code
  }
  el = null    // 主动释放el
}
```
#### 预期之外的输出
```javascript
var fn = function(){
  var arr = [];
  for(var i = 0; i < 10; i++){
    arr[i] = function(){
      console.log(i);
    }
  }
  
  for(var j = 0, len = arr.length; j < len; j++){
    arr[j]();
  }
}
```
![upload successful](/images/pasted-8.png)
我们想要的是从0，输出到9，最终输出的确实10次10。这段代码类似我们批量的对DOM元素绑定事件一样，很容易出错。
改成如下代码
```javascript
var fn = function(){
  var arr = [];
  for(var i = 0; i < 10; i++){
    (function(n){
      arr[i] = (function(){
        console.log(n);
      })
    })(i)
  }
  
  for(var j = 0, len = arr.length; j < len; j++){
    arr[j]();
  }
}
```
![upload successful](/images/pasted-10.png)
或者使用`let`修改代码
```javascript
var fn = function(){
  var arr = [];
  for(let i = 0; i < 10; i++){
    arr[i] = function(){
      console.log(i);
    }
  }
  
  for(var j = 0, len = arr.length; j < len; j++){
    arr[j]();
  }
}
```
![upload successful](/images/pasted-9.png)
使用`let`声名局部变量。不明白的可以看下`let`的作用域。