---
layout: post
title: ES7 Decorator 装饰者模式 浅出
category : 前端技术
tags : 前端 javascript
---


##ES6 已经来了，ES7 还会远么？

ES6 标准正式发布，众多新特性成为标准固然另人激动，然而更值得憧憬的还是未来。所以，让我们来看看 ES7（更正式的说法是 ES2016）有哪些激动人心的变化。最为人津津乐道的可能就是 async/await 了，不过我个人出对 decorator 模式有点感兴趣，本文尝试对此做一个介绍。

###decorator 是什么

ES7 的 decorator 概念是从 Python 借来的，在 Python 里，decorator 实际上是一个 wrapper，它作用于一个目标函数，对这个目标函数做一些额外的操作，然后返回一个新的函数：

```python
def my_decorator(fn):
  def inner(name):
    print 'Hello ' + fn(name)
  return inner

@my_decorator
def greet(name):
  return name

greet('Decorator!')
```

这种 @decorator 的写法其实是一种语法糖，从 `my_decorator`的定义就可以看出，它接收一个函数（ fn ）为参数，定义一个新的内部函数（ innner ），这个内部函数会定义一些行为，最后 `my_decorator` 返回这个内部函数（ inner ）。

上面的 `@my_decorator`等于：

```js
greet = my_decorator(greet)

greet('Decorator!')

```

## Hello Decorator!

ES7 中的 decorator 同样借鉴了这个语法糖，不过依赖于 ES5 的 Object.defineProperty 方法 。

##浅出
=====

想必大家都该玩过LOL吧。那么我就以英雄联盟英雄为例子。

##第一步：

* 英雄

```js
class Hero{
  constructor(def = 30,atk = 70,hp = 630){
    this.init({def,atk,hp});
  }
  init({def,atk,hp}){
    this.hp=hp;
    this.attcak=atk;
    this.defence=def;
  }
}
```

* 德玛西亚 ps：这只是个小德玛，简单的继承了英雄属性而已，没特色啊，防御和血量哪去了。。

```js
class 德玛西亚 extends Hero{
  init(args){
    super.init(args);
  }
  toString(){
    return `防御力:${this.defence},攻击力:${this.attcak},血量:${this.hp}`;
  }
}
var _德玛西亚 = new 德玛西亚();

console.log(`当前状态 ===> ${_德玛西亚}`);
// 当前状态 ===> 防御力:30,攻击力:70,血量:630
```

##第二步

* 初始化德玛属性

```js
function 德玛西亚初始(target, key, descriptor) {
  const method = descriptor.value;
  let moreHP = 150;
  let ret;
  descriptor.value = ({def,atk,hp})=>{
    hp+=moreHP;
    ret = target::method({def,atk,hp});
    return ret;
  }
  return descriptor;
}

class 德玛西亚 extends Hero{
  @德玛西亚初始;
  init(args){
    super.init(args);
  }
  toString(){
    return `防御力:${this.defence},攻击力:${this.attcak},血量:${this.hp}`;
  }
}
var _德玛西亚 = new 德玛西亚();

console.log(`当前状态 ===> ${_德玛西亚}`);
// 当前状态 ===> 防御力:30,攻击力:70,血量:780
```
血量变多了。还好吧，那下一步给他个被动技能吧！

####这里可能会有两个疑问：

1. `德玛西亚初始`方法的参数为啥是这三个？可以更换么？
2. `德玛西亚初始`方法为什么返回的是descriptor

####这里给出的解答作为参考：

>Decorators 的本质是利用了ES5的 Object.defineProperty 属性，这三个参数其实是和 Object.defineProperty参数一致的，因此不能更改。可以看看 bable转换后 的代码，其中有一句是descriptor = decorator(target, key, descriptor) || descriptor;，点到为止，这里不详细展开了，可自行看看这行代码的上下文

* 初始化德玛西亚的被动技能

```js


function 德玛被动() {
  return function(target){
    let extra = '(技能加成:盖伦在7秒内不受到任何伤害之后，每秒回复最大生命值的0.5% )';
    let method = target.prototype.toString;
    target.prototype.toString = (...args)=>{
      return target.prototype::method(args) + extra;
    }
    return target;
  }
}

@德玛被动()
class 德玛西亚 extends Hero{
  @德玛西亚初始;
  init(args){
    super.init(args);
  }
  toString(){
    return `防御力:${this.defence},攻击力:${this.attcak},血量:${this.hp}`;
  }
}
var _德玛西亚 = new 德玛西亚();

console.log(`当前状态 ===> ${_德玛西亚}`);
// 当前状态 ===> 防御力:30,攻击力:70,血量:780(技能加成:盖伦在7秒内不受到任何伤害之后，每秒回复最大生命值的0.5% )

```

好啦完整的德玛西亚出来了。


>代码直接放在 http://babeljs.io/repl/ 中运行查看结果，记得勾选Experimental选项和Evaluate选项

