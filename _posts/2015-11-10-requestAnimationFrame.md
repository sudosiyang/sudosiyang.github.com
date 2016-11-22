---
layout: post
title: requestAnimationFrame 详解
category : 前端技术
tags : 前端 javascript
---
浏览器中动画有两种实现形式：通过申明元素实现（如SVG中的元素）和脚本实现。

可以通过setTimeout和setInterval方法来在脚本中实现动画，但是这样效果可能不够流畅，且会占用额外的资源。可参考[《Html5 Canvas核心技术》](http://book.douban.com/subject/24533314/)中的论述：

它们有如下的特征：

1. 即使向其传递毫秒为单位的参数，它们也不能达到ms的准确性。这是因为javascript是单线程的，可能会发生阻塞。

2. 没有对调用动画的循环机制进行优化。

3. 没有考虑到绘制动画的最佳时机，只是一味地以某个大致的事件间隔来调用循环。

jQuery动画的实现考虑到兼容与易用性采用了setInterval来不断绘制新的属性值，从而达到动画的效果。

大部分浏览器的显示频率是16.7ms，由于浏览器的特性，setInterval会有一个丢帧的问题

即使向其传递毫秒为单位的参数，它们也不能达到ms的准确性。这是因为javascript是单线程的，可能会发生阻塞

jQuery会有一个全局设置```jQuery.fx.interval = 13 ```设置动画每秒运行帧数。

默认是13毫秒。该属性值越小，在速度较快的浏览器中（例如，Chrome），动画执行的越流畅，但是会影响程序的性能并且占用更多的 CPU 资源

那么归纳一点最关键的问题：

>开发着并不知道下一刻绘制动画的最佳时机是什么时候


```requestAnimationFrame``` 是专门为实现高性能的帧动画而设计的一个API

说简单点

setInterval、setTimeout是开发者主动要求浏览器去绘制，但是由于种种问题，浏览器可能会漏掉部分命令
requestAnimationFrame 就是浏览器什么要开始绘制了浏览器自己知道，通过requestAnimationFrame 告诉开发者，这样就不会出现重复绘制丢失的问题了
目前已在多个浏览器得到了支持，包括IE10+，Firefox，Chrome，Safari，Opera等，在移动设备上，ios6以上版本以及IE mobile 10以上也支持requestAnimationFrame，

唯一比较遗憾的是目前安卓上的原生浏览器并不支持requestAnimationFrame，不过对requestAnimationFrame的支持应该是大势所趋了，安卓版本的chrome 16+也是支持requestAnimationFrame的。


### 那么如何给动画设置帧数呢？

 

#### 最简单：

```js

var FPS = 60;

setInterval(draw, 1000/FPS);

```

这个简单做法，如果draw带有大量逻辑计算，导致计算时间超过帧等待时间时，将会出现丢帧。除外，如果FPS太高，超过了当时浏览器的重绘频率，将会造成计算浪费，例如浏览器实际才重绘2帧，但却计算了3帧，那么有1帧的计算就浪费了。

 

#### 成熟做法：

引入requestAnimationFrame，这个方法是用来在页面重绘之前，通知浏览器调用一个指定的函数，以满足开发者操作动画的需求。

这个函数类似setTimeout，只调用一次。

```js
function draw() { 
        requestAnimationFrame(draw); 
        // ... Code for Drawing the Frame ... 
}
``` 

递归调用，就可以实现定时器。

但是，这样完全跟浏览器帧频同步了，无法自定义动画的帧频，是无法满足需求的。

 

接下来需要考虑如何控制帧频。

#### 简单做法：

```js
var fps = 30;
function tick() {
　　setTimeout(function() {
　　　　requestAnimationFrame(tick);
　　　　draw(); // ... Code for Drawing the Frame ...
　　}, 1000 / fps);
}
tick();
```
 

这种做法，比较直观的可以发现，每一次setTimeout执行的时候，都还要再等到下一个requestAnimationFrame事件到达，累积下去会造成动画变慢。

 

#### 自行控制时间跨度：

```js
var fps = 30;
var now;
var then = Date.now();
var interval = 1000/fps;
var delta;

function tick() {
　　requestAnimationFrame(tick);
　　now = Date.now();
　　delta = now - then;
　　if (delta > interval) {
　　　　// 这里不能简单then=now，否则还会出现上边简单做法的细微时间差问题。
　　　　// 例如fps=10，每帧100ms，而现在每16ms（60fps）执行一次draw。16*7=112>100，需要7次才实际绘制一次。
　　　　// 这个情况下，实际10帧需要112*10=1120ms>1000ms才绘制完成。
　　　　then = now - (delta % interval);
　　　　draw(); // ... Code for Drawing the Frame ...
　　}
}
tick();
```
 

 

#### 针对低版本浏览器再优化：

如果浏览器没有requestAnimationFrame函数，实际底层还只能用setTimeout模拟，上边做的都是无用功。那么可以再改进一下。

```js
var fps = 30;
var now;
var then = Date.now();
var interval = 1000/fps;
var delta;
window.requestAnimationFrame = window.requestAnimationFrame || window.mozRequestAnimationFrame || window.webkitRequestAnimationFrame || window.msRequestAnimationFrame;

function tick() {
　　if(window.requestAnimationFrame)
   {
　　    requestAnimationFrame(tick);
　　    now = Date.now();
　　    delta = now - then;
　　    if (delta > interval) {
        // 这里不能简单then=now，否则还会出现上边简单做法的细微时间差问题。例如fps=10，每帧100ms，而现在每16ms（60fps）执行一次draw。16*7=112>100，需要7次才实际绘制一次。这个情况下，实际10帧需要112*10=1120ms>1000ms才绘制完成。
　　　　    then = now - (delta % interval);
　　　　    draw(); // ... Code for Drawing the Frame ...
　　    }
   }
   else
   {
       setTimeout(tick, interval);
　　　　draw();
   }
}
tick();
``` 



最后，还可以加上暂停。

```js
var fps = 30;
var pause = false;
var now;
var then = Date.now();
var interval = 1000/fps;
var delta;
window.requestAnimationFrame = window.requestAnimationFrame || window.mozRequestAnimationFrame || window.webkitRequestAnimationFrame || window.msRequestAnimationFrame;

function tick() {
     if(pause)
          return;
 　　if(window.requestAnimationFrame)
     {
　　　　...
     }
     else
     {
          ...
     }
}
tick();
```
