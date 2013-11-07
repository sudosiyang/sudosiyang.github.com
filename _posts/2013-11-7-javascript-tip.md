---
layout: post
title: javascript 小技巧
category : 前端技术
tags : 前端 杂谈
---
===


**字符串逆序**

```javascript

var reverse=function(str){
    str.split("").reverse().join('');
}
```


**去掉前后空格**

```javascript

var trim=function(arr){
    return arr.replace(/(^\s*)|(\s*$)/g, "");
}
```


**数组复制**

```javascript

var copy=function(arr){
    return arr.slice(0);
}
```
**将伪数组转化为数组**

```javascript

var makeArray = function(obj){
    return Array.prototype.slice.call(obj,0);
}
```