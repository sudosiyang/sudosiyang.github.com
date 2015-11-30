---
layout: post
title: ES6 实现几种 设计模式
category : 前端技术
tags : 前端 javascript
---

最近将部分项目从ES5转换到了ES6,从新的语法中体会到了很多。无论是可读性还是代码量方面都有很大改善。既然引入了es6，那设计模式的实现方法就需要重新写一遍了。

###单例模式


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

###工厂模式

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


###代理模式


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

###命令模式

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

###迭代模式

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

###装饰者模式

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

先介绍这么多。。有时间在更新吧！
