---
layout: post
title: Nodejs-多进程、多线程和分布式
category : 前端技术
tags : 前端 JavaScript nodejs
---

# 性能实践

## 避免使用 Lodash

- 使用像 lodash 这样的方法库这会导致不必要的依赖和较慢的性能
- 随着新的 V8 引擎和新的 ES 标准的引入，原生方法得到了改进，现在性能比方法库提高了 50%

使用 ESLint 插件检测：

```js
{
  "extends": [
    "plugin:you-dont-need-lodash-underscore/compatible"
  ]
}
```

## benchmark

```js
const _ = require('lodash'),
  __ = require('underscore'),
  Suite = require('benchmark').Suite,
  opts = require('./utils');
  //cf. https://github.com/Berkmann18/NativeVsUtils/blob/master/utils.js

const concatSuite = new Suite('concat', opts);
const array = [0, 1, 2];

concatSuite.add('lodash', () => _.concat(array, 3, 4, 5))
  .add('underscore', () => __.concat(array, 3, 4, 5))
  .add('native', () => array.concat(3, 4, 5))
  .run({ 'async': true });
```

## 使用 prof 进行性能分析

- 使用 tick-processor 工具处理分析

```sh
node --prof profile-test.js
```

```sh
npm install tick -g

node-tick-processor
```


## V8 内存管理与垃圾回收

V8 的内存分为 New Space 和 Old Space，New Space 的大小默认为 8M，Old Space 的大小默认为 0.7G，64位系统这两个数值翻倍

* 首先进入 New Space，New Space 被平均分为两份，每次 GC 都会将一份中的活着的对象复制到另一份，所以它的空间使用率是 50%，这个算法叫做 Cheney 算法，这个操作叫做 Scavenge
* 过一段时间，如果 New Space 中的对象还活着，会被挪到 Old Space 中去，GC 会每隔一段时间遍历 Old Space 中死掉的对象，然后整理碎片
* 如果缓存增加（比如使用对象缓存了很多用户信息），GC 是不知道这些缓存死了还是活着的，他们会不停地查看这个对象，以及这个对象中的子对象是否还存活，如果这个对象数特别大，那么 GC 遍历的时间也会特别长
* 当我们进行密集型计算的时候，会产生很多中间变量，这些变量往往在 New Space 中就死掉了，那么 GC 也会在这里多次地进行 New Space 区域的垃圾回收

## 分析 GC 日志

* 内存暴涨，尤其是 Old Space 内存的暴涨，会直接导致 GC 的次数和时间增长
* 缓存增加，导致 GC 的时间增加，无用遍历过多
* 密集型计算，导致 GC Now Space次数增加


## 使用 headdump 堆快照

- 代码加载模块进行快照文件生成
- Chrome Profiles 加载快照文件

```sh
yarn add heapdump -D
```

```js
const heapdump = require('heapdump');
const string = '1 string to rule them all';

const leakyArr = [];
let count = 2;
setInterval(function () {
  leakyArr.push(string.replace(/1/g, count++));
}, 0);

setInterval(function () {
  if (heapdump.writeSnapshot()) console.log('wrote snapshot');
}, 20000);
```