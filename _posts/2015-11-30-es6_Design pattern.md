---
layout: post
title: ES6 实现几种 设计模式
category : 前端技术
tags : 前端 javascript
---

最近将部分项目从ES5转换到了ES6,从新的语法中体会到了很多。无论是可读性还是代码量方面都有很大改善。既然引入了es6，那设计模式的实现方法就需要重新写一遍了。

### 单例模式


```js
'use strict';
let __instance = (function () {
  let instance;
  return (_instance) => {
    if (_instance) instance = _instance;
    return instance;
  }
})();

class Singleton {
  constructor() {
    if (__instance()) return __instance();
    // todo init
    __instance(this);
  }
}
console.log(new Singleton() == new Singleton())
```

### 工厂模式

```js
'use strict';
class Car{
  constructor(){
    this.door=4;
  }
  static fatcory(type){
      return new Car[type]();
  }
}
Car.Trunk = class Trunk extends Car{
    constructor(){
      super();
      this.door=2
    }
}

console.log(Car.fatcory("Trunk").door)
```


### 代理模式

> 使用代理模式创建代理对象，让代理对象控制目标对象的访问(目标对象可以是远程的对象、创建开销大的对象或需要安全控制的对象),并且可以在不改变目标对象的情况下添加一些额外的功能。

```js
'use strict';
class Real {
  toDo() {
    console.log(`to do something...`);
  }
}

class Proxy extends Real {
  constructor() {
    super();
  }

  toDo() {
    setTimeout(super.toDo, 1000 * N);
  }
}

new Proxy().toDo(); //N s后, to do...

```


### 迭代模式

> 提供一种方法顺序访问一个聚合对象中的各种元素，而又不暴露该对象的内部表示

```js
'use strict';
let array = {
  data: [1, 2, 3, 4, 5],
  [Symbol.iterator](){
    let index = 0;
    return {
      next: () => {
        if (index < this.data.length) return {value: this.data[index++], done: false};
        return {value: undefined, done: true};
      },
      hasNext: () => index < this.data.length,
      rewind: () => index = 0,
      current: () => {
        index -= 1;
        if (index < this.data.length) return {value: this.data[index++], done: false};
        return {value: undefined, done: true};
      }
    }
  }
};

// for...of
for (let val of array) {
  console.log(val);
}
```

### 装饰者模式

>在不改变原类和继承的情况下动态扩展对象功能，通过包装一个对象来实现一个新的具有原对象相同接口的新的对象

```js
'use strict';
class Equipments{
  static loricae(hero){
    hero.defence += 15;
    hero.hp += 200;
  }
  static glaive(hero){
    hero.attcak += 40;
  }
}

class Hero {
  constructor({def = 30,atk = 70,hp = 630} = {}) {
    this.hp = hp;
    this.attcak = atk;
    this.defence = def;
  }
  
  toString() {
    return `防御力:${this.defence},攻击力:${this.attcak},血量:${this.hp}`;;
  }
}

let hero = new Hero();
console.log(hero); //{"hp":630,"attcak":70,"defence":30}

Equipments.loricae(hero)
console.log(hero); //{"hp":830,"attcak":70,"defence":45}

Equipments.glaive(hero)
console.log(hero); //{"hp":830,"attcak":110,"defence":45}
```


### 订阅/发布模式

>发布-订阅（publish–subscribe）是一种消息传播模式，消息的发送者（发布者）不会将消息直接发送给特定的接收者（订阅者）。而是将发布的消息按特征分类，无需对订阅者（如果有的话）有所了解。同样的，订阅者可以表达对一个或多个类别的兴趣，只接收感兴趣的消息，无需对发布者（如果有的话）有所了解。

```js
'use strict';
class Event {
  constructor() {
    this.subscribers = new Map([['any', []]]);
  }

  on(type = 'any',fn) {
    let subs = this.subscribers;
    if (!subs.get(type)) return subs.set(type, [fn]);
    subs.set(type, (subs.get(type).push(fn)));
  }

  emit(type = 'any', ...agrs) {
    for (let fn of this.subscribers.get(type)) {
      fn(...agrs);
    }
  }
}

let event = new Event();

event.on("click",(e) => console.log(`click Dom: ${e.target}`));
event.emit('click',{target:"div"}); //click Dom: div
```



### 命令模式
> 命令模式将一个请求或者操作封装到一个对象中。命令模式将发出命令和执行命令的责任分割开，委派给不同的对象。命令模式也允许请求方个接受方相互独立，使得请求方不需要知道接收方的接口等。

e.g.在富文本编辑器中:

```js
'use strict';
class Cmd{
  constructor(){
    this.execList=[];
    this.cmds={
      blod:function(){},
      font_family:function(){},
      font_color:function(){},
      code:function(){}
      //等等
      };//命令集合
  }
  exec(cmd,...args){
    let command = this.cmds[cmd];
    if(command){
      this.execList.push({
        cmd:cmd,
        args:args
      });
      command(args)
    }
    
  }
  
  undo(){
    let handle = this.execList.pop();
    console.log(handle)
    //do something;
  }
}

let cmd=new Cmd()
cmd.exec("blod");
cmd.exec("code","let a;");
cmd.undo();
```

先介绍这么多。。有时间在更新吧！
