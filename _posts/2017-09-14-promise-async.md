---
layout: post
title: Promise原理分析及手写
category : 前端技术
tags : 前端 JavaScript
---

分析`Promise`实现有利于理解异步函数调用过程和链式调用的实现.

最简单的`Promise`实现有7个主要属性, state(状态), value(成功返回值), reason(错误信息), resolve方法, reject方法, then方法.

# 简单代码实现

```js
class Promise{
  constructor(executor) {
    this.state = 'pending';
    this.value = undefined;
    this.reason = undefined;
    let resolve = value => {
      if (this.state === 'pending') {
        this.state = 'fulfilled';
        this.value = value;
      }
    };
    let reject = reason => {
      if (this.state === 'pending') {
        this.state = 'rejected';
        this.reason = reason;
      }
    };
    try {
      // 立即执行函数
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }
  then(onFulfilled, onRejected) {
    if (this.state === 'fulfilled') {
      let x = onFulfilled(this.value);
    };
    if (this.state === 'rejected') {
      let x = onRejected(this.reason);
    };
  }
}
```
> 简单代码实现基本看代码就知道.



> 下面主要分析下复杂代码的实现.

复杂代码实现:

```js
class Promise{
  constructor(executor) {
    this.state = 'pending';
    this.value = undefined;
    this.reason = undefined;
    this.onResolvedCallbacks = [];
    this.onRejectedCallbacks = [];
    let resolve = value => {
      if (this.state === 'pending') {
        this.state = 'fulfilled';
        this.value = value;
        this.onResolvedCallbacks.forEach(fn=>fn());
      }
    };
    let reject = reason => {
      if (this.state === 'pending') {
        this.state = 'rejected';
        this.reason = reason;
        this.onRejectedCallbacks.forEach(fn=>fn());
      }
    };
    try {
      // 立即执行函数
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }
  then(onFulfilled, onRejected) {
    // onFulfilled如果不是函数，就忽略onFulfilled，变成默认直接返回value的函数
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
    // onRejected如果不是函数，就忽略onRejected，变成默认直接扔出错误的函数
    onRejected = typeof onRejected === 'function' ? onRejected : err => { throw err };

    // 声明返回的promise2
    let promise2 = new Promise((resolve, reject) => {
      
      if (this.state === 'pending') {
        this.onResolvedCallbacks.push(() => {
          let x = onFulfilled(this.value);
          resolvePromise(promise2, x, resolve, reject);
        })
        this.onRejectedCallbacks.push(() => {
          let x = onRejected(this.reason);
          resolvePromise(promise2, x, resolve, reject);
        })
      };

      if (this.state === 'fulfilled') {
        let x = onFulfilled(this.value);
        // resolvePromise函数，处理自己return的promise和默认的promise2的关系
        resolvePromise(promise2, x, resolve, reject);
      };
      if (this.state === 'rejected') {
        let x = onRejected(this.reason);
        resolvePromise(promise2, x, resolve, reject);
      };

    });
    // 返回promise，完成链式
    return promise2;
  }
  catch(fn){
    return this.then(null,fn);
  }
}
```

`Promise`的构造函数属性:

state(状态)属性: pending, fulfilled, rejected.

value运行成功后返回的变量.

reason运行错误后存储的错误信息.

resolve方法:

1. state(状态)在pending的情况下, 把state变成fulfiled(防止在其他状态下改变, 实现只能改变一次状态);
2. 把value变量存储起来, 供以后then调用.
3. 把存储在onResolvedCallbacks数组(队列)里的方法都运行一遍.

reject方法:

1. state(状态)在pending的情况下, 把state变成rejected(防止在其他状态下改变, 实现只能改变一次状态);
2. 把reason变量存储起来, 供以后then调用.
3. 把存储在onRejectedCallbacks数组(队列)里的方法都运行一遍.

`onResolvedCallbacks`

为什么需要onResolvedCallbacks和onRejectedCallbacks存储then方法的队列?

在不是链式调用的情况下, then方法可以多次被调用, 因此需要两个数组分别储存任务队列.

例如:

```js
let p = new Promise((s, r) => {
  s('resoved.');
})
p.then(()=>{
  console.log('Run then');
})
p.then(()=>{
  console.log('2nd run then');
})
```

executor构造函数的立即执行方法, 参数包括resolve和reject方法.

# Promise实例属性

then方法包括两个函数参数onFulfilled和onRejected, 用于分别接收成功和失败后的处理方法.

例如:

```js
p.then(
  (data) =>{
    console.log('Success:', data);
  },
  error => {
    console.log('Fail:', error);
  }
);
```

由于需要像下面代码一样实现链式调用

```js
p
  .then(()=>{console.log('Success.')})
  .then(()=>{console.log('and then.')})
```

then需要返回一个新的Promise实例, 并继续调用新实例的then方法形成链式.

理解起来的代码是这样的:

```js
(p.then()).then()
```

创建新Promise实例的executor立即执行以下逻辑:

如果state是pending,

1. 把onFulfilled方法(即实例then的处理成功方法)放到onResolvedCallbacks数组队列里.
2. 把onRejected方法(即实例then的处理失败方法)放到onRejectedCallbacks数组队列里.

如果state是fulfilled, 则代入value立即执行onFulfilled方法.

如果state是rejected, 则代入value立即执行onRejected方法.

如果在onFulfilled和onRejected方法里返回了新的Promise实例需要继续往下执行, 那么onFulfilled和onRejected的返回结果放到resolvePromise();判断并处理.

# 分析resolvePromise

resolvePromise 代码实现

```js
function resolvePromise(promise2, x, resolve, reject){
  // 循环引用报错
  if(x === promise2){
    // reject报错
    return reject(new TypeError('Chaining cycle detected for promise'));
  }
  // 防止多次调用
  let called;
  // x不是null 且x是对象或者函数
  if (x != null && (typeof x === 'object' || typeof x === 'function')) {
    try {
      // A+规定，声明then = x的then方法
      let then = x.then;
      // 如果then是函数，就默认是promise了
      if (typeof then === 'function') {
        // 就让then执行 第一个参数是this   后面是成功的回调 和 失败的回调
        then.call(x, y => {
          // 成功和失败只能调用一个
          if (called) return;
          called = true;
          // resolve的结果依旧是promise 那就继续解析
          resolvePromise(promise2, y, resolve, reject);
        }, err => {
          // 成功和失败只能调用一个
          if (called) return;
          called = true;
          reject(err);// 失败了就失败了
        })
      } else {
        resolve(x); // 直接成功即可
      }
    } catch (e) {
      // 也属于失败
      if (called) return;
      called = true;
      // 取then出错了那就不要在继续执行了
      reject(e); 
    }
  } else {
    resolve(x);
  }
}
```

**resolvePromise 代码分析**

让`p = new Promise()`, `p.then(rs, rj)`返回了一个新的`Promise`实例(假设为p2), 可以让`p2.then(rs2, rj2)`继续执行...

rs运行后得到的结果为x(一般没有返回值, 所以为undefined), x代入resolvePromise()方法检验.

1. 如果x是普通的值, 立刻执行resolve方法处理队列.
2. 如果x有属性then并且then是函数, 那个可以被认为x是Promise, 这种情况针对的是then里嵌套并返回新的Promise实例.
3. 考虑下面这种情况, then里嵌套返回自身, then继续call自身然后返回自身, 进入死循环.

```js
var p2 = p.then(data => {
  return p2;
})
```

为了避免这种情况, 必须在resolvePromise判断是否是自身, 解决循环引用问题.

实例运行分析

```js
const test = function() {
  return new Promise((res, rej)=>{
    setTimeout(()=>{
      console.log('ok');
      res();
    }, 2000)
  })
};
test().then(()=>{console.log(`1st then`)}).then(()=>{console.log(`2nd then`)});

```
1. 当定时器结束时, 输出'ok', 执行了第一个Promise实例的resolve, 第一个Promise实例的state状态变成fulfilled;
2. 第一个Promise实例的onResolvedCallbacks队列执行任务
  * 完成第一个then的onFulfilled方法, 并把返回的结果放到resolvePromise里执行, 这时的resolve和reject函数是第二个Promise实例的(即then里的Promise实例);
  ```js
  let x = onFulfilled(this.value);
  resolvePromise(promise2, x, resolve, reject);
  ```
  * resolvePromise方法如果遇到普通值则执行第二个Promise实例的resolve方法, 将第二个Promise的状态从pending转为fulfilled, 然后执行第二个Promise实例的onResolvedCallbacks队列.
  * 第二个Promise实例存在调用then, 因此它的onResolvedCallbacks队列里存放着第三个Promise的resolve和reject方法并等待着第二个Promise实例的fulfilled状态改变(即resolve被调用).
  * 因为没有第三个then, 因此第三个Promise实例的onResolvedCallbacks队列没有任务, 调用链到此为止.

# 其他Promise静态方法

```js
//resolve方法
Promise.resolve = function(val){
  return new Promise((resolve,reject)=>{
    resolve(val)
  });
}
//reject方法
Promise.reject = function(val){
  return new Promise((resolve,reject)=>{
    reject(val)
  });
}
//race方法 
Promise.race = function(promises){
  return new Promise((resolve,reject)=>{
    for(let i=0;i<promises.length;i++){
      promises[i].then(resolve,reject)
    };
  })
}
//all方法(获取所有的promise，都执行then，把结果放到数组，一起返回)
Promise.all = function(promises){
  let arr = [];
  let i = 0;
  function processData(index,data){
    arr[index] = data;
    i++;
    if(i == promises.length){
      resolve(arr);
    };
  };
  return new Promise((resolve,reject)=>{
    for(let i=0;i<promises.length;i++){
      promises[i].then(data=>{
        processData(i,data);
      },reject);
    };
  });
}
```