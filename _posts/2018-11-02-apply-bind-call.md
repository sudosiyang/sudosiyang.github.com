---
layout: post
title: call、apply、bind以及如何实现
category : 前端技术
tags : 前端 JavaScript
---


# call

语法：

```js
fun.call(thisArg[, arg1[, arg2[, ...]]])
```

thisArg：fun函数运行时指定的this值，可能的值为：

* 不传，或者传null，undefined， this指向window对象
* 传递另一个函数的函数名fun2，this指向函数fun2的引用
值为原始值(数字，字符串，布尔值),this会指向该原始值的自* 动包装对象，如 String、Number、Boolean
* 传递一个对象，函数中的this指向这个对象

例如：

```js
function a(){
    console.log(this);
}
function b(){}

a.call(b);  // function b(){}

```

经常会看到这种使用情况：

```js
function list() {
  // 将arguments转成数组
  return Array.prototype.slice.call(arguments);  
}
list(1,2,3);  // [1, 2, 3]

```

为什么能实现这样的功能将arguments转成数组？首先call了之后，this指向了所传进去的arguments。我们可以假设slice方法的内部实现是这样子的：创建一个新数组，然后for循环遍历`this`，将`this[i]`一个个地赋值给新数组，最后返回该新数组。因此也就可以理解能实现这样的功能了。

# apply

语法：
```js
// Chrome 14 以及 Internet Explorer 9 仍然不接受类数组对象。
// thisArg的可能值和call一样
fun.apply(thisArg[, argsArray])
```

例如：

var numbers = [5, 6, 2, 3, 7];
var max = Math.max.apply(null, numbers);
console.log(max)  // 7
平时Math.max只能这样子用：Math.max(5,6,2,3,7);
利用apply的第二个参数是数组的特性，从而能够简便地从数组中找到最大值。

# bind

基本用法

语法：

```js
fun.bind(thisArg[, arg1[, arg2[, ...]]]);
```

bind()方法会创建一个新函数，称为绑定函数。

bind是ES5新增的一个方法，不会执行对应的函数（call或apply会自动执行对应的函数），而是返回对绑定函数的引用。

当调用这个绑定函数时，thisArg参数作为 this，第二个以及以后的参数加上绑定函数运行时本身的参数按照顺序作为原函数的参数来调用原函数。

简单地说，bind会产生一个新的函数，这个函数可以有预设的参数。

```js
function list() {
  // 将arguments转成数组
  return Array.prototype.slice.call(arguments);  
}

var leadingThirtysevenList = list.bind(undefined, 37); // 绑定函数
var list = leadingThirtysevenList(1, 2, 3); // 调用绑定函数

console.log(list); // [37, 1, 2, 3]
```
# bind调用简单

把类数组换成真正的数组，bind能够更简单地使用：

## apply用法

```js
var slice = Array.prototype.slice;
// ...
slice.apply(arguments);  // 类似对象的方法那样调用
```

## bind用法

```js
var unboundSlice = Array.prototype.slice;
var slice = Function.prototype.apply.bind(unboundSlice);
// ...
slice(arguments);  // 直接调用，简单
```

# 它们的区别

相同之处：改变函数体内 this 的指向。

不同之处：

* call、apply的区别：接受参数的方式不一样。
* bind：不立即执行。而apply、call 立即执行。


# 实现Call

```js
//传递参数从一个数组变成逐个传参了,不用...扩展运算符的也可以用arguments代替
Function.prototype.myCall = function (context, ...args) {
    //这里默认不传就是给window,也可以用es6给参数设置默认参数
    context = context || window
    args = args ? args : []
    //给context新增一个独一无二的属性以免覆盖原有属性
    const key = Symbol()
    context[key] = this
    //通过隐式绑定的方式调用函数
    const result = context[key](...args)
    //删除添加的属性
    delete context[key]
    //返回函数调用的返回值
    return result
}
```

# 实现apply

```js
Function.prototype.myApply = function (context, args) {
    //这里默认不传就是给window,也可以用es6给参数设置默认参数
    context = context || window
    args = args ? args : []
    //给context新增一个独一无二的属性以免覆盖原有属性
    const key = Symbol()
    context[key] = this
    //通过隐式绑定的方式调用函数
    const result = context[key](...args)
    //删除添加的属性
    delete context[key]
    //返回函数调用的返回值
    return result
}
```


# 实现bind

bind和apply的区别在于,bind是返回一个绑定好的函数,apply是直接调用.其实想一想实现也很简单,就是返回一个函数,里面执行了apply上述的操作而已.不过有一个需要判断的点,因为返回新的函数,要考虑到使用new去调用,并且new的优先级比较高,所以需要判断new的调用,还有一个特点就是bind调用的时候可以传参,调用之后生成的新的函数也可以传参,效果是一样的,所以这一块也要做处理
因为上面已经实现了apply,这里就借用一下,实际上不借用就是把代码copy过来

```js
Function.prototype.myBind = function (context, ...args) {
    const fn = this
    args = args ? args : []
    return function newFn(...newFnArgs) {
        if (this instanceof newFn) {
            return new fn(...args, ...newFnArgs)
        }
        return fn.apply(context, [...args,...newFnArgs])
    }
}
```