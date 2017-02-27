---
layout: post
title: ES6proxy实践（译）
category : 前端技术
tags : 前端 javascript 译文
---

一个proxy是一个包装对象或包装的项目功能和监测器，又名对象目标。我们使用proxy来阻止对目标函数或对象的直接访问。
proxy对象有一些方法，处理目标的访问。方法与Reflect API中使用的方法相同。因此，本节假设您熟悉Reflect API，因为您将使用相同的调用。提供以下方法：

*	apply
*	construct
*	defineProperty
*	deleteProperty
*	get
*	getOwnPropertyDescriptor
*	getPrototypeOf
*	has
*	isExtensible
*	ownKeys
*	preventExtensions
*	set
*	setPrototypeOf

一旦proxy对象的方法被执行，我们可以运行任何代码，即使没有访问目标。proxy决定是否要为您提供对目标的访问权限，或者自己处理请求。

可用的方法允许您监视几乎任何用例。一些行为不能被困住。

我们不能知道我们的对象是否与另一个值相比较，或者被另一个对象包装。我们不能知道我们的对象是否是操作符的操作数。实际上，没有这些用例几乎没有问题。

#####定义proxy

我们可以通过以下方式创建proxy：

```js
let = new Proxy( target, trapObject );
```
第一个参数表示targetproxy的构造函数，类或对象。
第二个参数是一个包含方法的对象，一旦以特定方式使用proxy，就会执行该方法。

让我们通过proxy以下目标来实践我们的知识：

```js
class Student {
    constructor(first, last, scores) {
        this.firstName = first;
        this.lastName = last;
        this.testScores = scores;
    }
    get average() {
        let average = this.testScores.reduce( 
            (a,b) => a + b, 
            0 
        ) / this.testScores.length;
        return average;
    }
}
let john = new Student( 'John', 'Dwan', [60, 80, 80] );
```

我们现在将在目标对象上定义一个proxyjohn：

```js
let john= new Proxy( john, {
    get: function( target, key, context ) {
        console.log( john[${key}] was accessed.);
        // return undefined;
    } 
});
```

该get每当我们试图访问一个属性执行proxy的方法。

```js
johnProxy.getGrade
> john[getGrade] was accessed.
> undefined
```

```js
johnProxy.testScores
> john[testScores] was accessed.
> undefined
```

上面定义的gettrap选择忽略要返回的目标对象的所有字段值undefined。
让我们定义一个稍微更有用的proxy，允许访问averagegetter函数，但返回undefined其他任何东西：

```js
let johnMethod= new Proxy( john, {
    get: function( target, key, context ) {
        if ( key === 'average' ) {
            return target.average;
        }
    } 
});
johnMethodProxy.firstName
undefined
johnMethodProxy.average 
73.33333333333333
```

我们可以得出结论，proxy可以用于的目的

* 定义访问修饰符，
* 通过对象的公共接口提供验证。

proxy的目标也可以是函数。

```js
    let factorial = n =>
    n <= 1 ? n : n * factorial( n - 1 );
```
例如，你可能想知道有多少次factorial调用函数表达式factorial( 5 )。这是一个自然的问题，要求制定一个自动化测试。
让我们定义一个proxy来找出答案。

```js
factorial = new Proxy( factorial, {
   apply: function( target, thisValue, args ) {
        console.log( 'I am called with', args );
        return target( ...args );
   } 
});
 
factorial( 5 );
> I am called with [5]
> I am called with [4]
> I am called with [3]
> I am called with [2]
> I am called with [1]
> 120

```

我们添加了一个proxy，捕获方法的所有调用factorial，包括递归调用。我们将proxy引用等同于factorial引用，以便所有递归函数调用都被proxy。原来的阶乘函数的引用是通过访问targetproxy服务器内部的说法。
我们现在对代码做两个小的修改。
首先，我们将更换

```js
return target( ...args );
```

与

```js
return Reflect.apply( 
    target, 
    thisValue, 
    args );
```

我们可以在proxy中使用Reflect API。除了演示Reflect API的使用之外，没有真正的替换原因。
二，而不是记录，现在我们将计算在函数调用的数量numOfCalls变量。
如果您在开发人员工具中执行这段程式码，请务必重新载入网页，因为现有程式码可能会与此互动

```js
let factorial = n =>
    n <= 1 ? n : n * factorial( n - 1 );
 
let originalFactorial = factorial;
let numOfCalls = 0;
factorial = new Proxy( factorial, {
   apply: function( target, thisValue, args ) {
        numOfCalls += 1;
        return Reflect.apply(
            target, 
            thisValue, 
            args 
        );
   } 
});
 
factorial( 5 ) && numOfCalls;
> 5

```
#####可撤销proxy
我们还可以使用创建可撤销proxyProxy.revocable。这在我们将proxy传递给其他对象时非常有用，但是我们希望集中控制何时关闭我们的proxy。

```js
let payload = {
    website: 'zsoltnagy.eu',
    article: 'Proxies in Practice',
    viewCount: 15496
}
 
let revocable = Proxy.revocable( payload, {
   get: function( ...args ) {
        console.log( 'Proxy' );
        return Reflect.get( ...args );
   } 
});
 
let = revocable.proxy;
 
proxy.website
> Proxy
> "zsoltnagy.eu"
 
revocable.revoke();
 
proxy.website
> Uncaught TypeError: Cannot perform 'get' on a that 
> has been revoked
>    at <anonymous>:3:6

```
一旦我们撤销了proxy，它会抛出一个错误，当我们尝试使用它。
作为revoke方法和proxy内部访问revocable，我们可以使用对象的ES6简写符号缩短我们的代码：

```js	 
// ...
// Create a revocable proxy
let {proxy, revoke} = Proxy.revocable( payload, {
   get: function( ...args ) {
        console.log( 'Proxy' );
        return Reflect.get( ...args );
   } 
});
// Revoke the proxy
revoke();
```
#####用例
在学习Proxy.revocable时，我们得出结论，我们可以通过可撤销proxy集中控制数据访问。我们需要做的是将可撤销的proxy对象传递给其他消费者对象，并在我们要使数据不可访问时撤销它们的访问。
在练习1中，您可以构建一个计数方法被调用的次数的proxy。proxy可用于自动测试。事实上，如果你阅读SinonJs文档，你可以看到proxy的多个用例。
您还可以构建一个虚假的服务器进行开发，拦截一些API调用并使用静态JSON文件来回答它们。对于未开发的API调用，proxy只是简单地传递请求，并让客户端与真实服务器交互。
在练习2中，你的任务是在着名的Fibonacci问题上构建一个memoizationproxy，并且我们将检查我们用查找保存的函数调用。如果您扩展此构思，您还可以使用proxy来记住昂贵的API调用的响应，并在不重新计算结果的情况下提供这些调用。
练习3突出了三个用例。
首先，我们可能需要处理具有非限制结构的JSON数据。如果我们不能对文档的结构做出假设，proxy就会派上用场。
第二，我们可以使用proxy建立一个日志层，包括日志，警告消息和错误。
第三，如果我们能够控制当我们记录错误时，我们也可以使用proxy来控制客户端验证。
如果我们结合的思想练习2和练习3中，我们可以使用proxy服务器来限制访问基于用户凭据端点。显然，这种方法不会保存服务器端的实现，但proxy给你一个方便的方式来处理访问权限。
#####练习
**练习1**.假设fibonacci给出以下实现：

```js
fibonacci = n => 
    n <= 1 ? n : 
    fibonacci( n - 1 ) + fibonacci( n - 2 );
```
确定fibonacci在评估时调用函数的次数fibonacci( 12 )。
确定有多少次fibonacci被调用，参数2评测的时候fibonacci( 12 )。
________________________________________

**练习2**.创建构建的查找表中的proxyfibonacci呼叫，记忆先前计算的值。任何n，如果该值fibonacci( n )已经被前计算，使用查找表，并返回而不是执行递归调用对应的值。
使用这个proxy，并确定多少次fibonacci函数被调用

• 总共

• 与参数 2
同时评估fibonacci( 12 )。
________________________________________

**练习3**.假设的对象payload中给出。您不能对有效载荷的结构做任何假设。想象一下，它的内容来自服务器，API规范可能会有所不同。
创建在每当在不存在的代码中访问属性时执行以下语句的proxy：

```js
console.error(' Error:'+ payload[${key}] +'does not exist. '); 
```

您不能修改对的引用payload，您必须使用原始payload引用来访问其字段。

________________________________________

**练习**4.给定对象

```js
let course = {
    name: 'ES6 in Practice',
    _price: 99,
    currency: '€',
    get price() {
        return `${this._price}${this.currency}`;
    }
};
```
定义可撤销的proxy，在5分钟（或300.000毫秒）的持续时间内为您提供价格的90％折扣。5分钟后撤销折扣。
原始course对象应始终提供对原始价格的访问。


译自与：[http://www.zsoltnagy.eu/es6-proxies-in-practice/](http://www.zsoltnagy.eu/es6-proxies-in-practice/)