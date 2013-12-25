---
layout: post
title:  JavaScript 跨域访问的问题和解决过程
category : 前端技术
tags : 前端 设计
---
===
分享一下最近用jQuery跨域请求的经历，希望能给大家一些关于这个方案的概念和资料。

1.代码很简单

```javascript

$.ajax({
	url: 'http://www.tetequ.com',
	type: 'GET'
}).done(function() {
	console.log("success");
}).fail(function() {
	console.log("error");
}).always(function() {
	console.log("complete");
});

```

上去运行一下......
当然不能用!什么都还没做呢，就想做跨域访问这么危险的事情,嘿嘿！

下面是Chrome给出的错误提示 

![网页设计](/blog-assets/2013-12-25/1.png)

2.在服务器端做点手脚(php为例)

```php

header("Access-Control-Allow-Origin:*");// 可以设置为详细的地址


```

3.好了现在Chrome中的Get已经可以运行了。

4.还有问题。。。。忽然发现在IE8和IE9中无法运行，而在其他的浏览器中都正常（opera未测试，google说这个浏览器也有问题。不过这东西比较小众）
使用Fiddler发现 这个动作根本没有被提交到服务器端。。。。
这是多么痛的领悟啊。（IE 为什么每次你都这么另类！，jQuery你为什么不兼容ie8和ie9的跨域提交功能。。加点代码很麻烦么！！！）

最后用各种方法知道了---

IE8以上的版本跨域提交需要使用XDomainRequest 对象。。。。关于 XDomainRequest 请在这里查看详细，<http://msdn.microsoft.com/en-us/library/cc288060(VS.85).aspx>

解决代码如下：

```javascript

var xdr = new XDomainRequest();
        xdr.onload = function (e) {
            var data = $.parseJSON(xdr.responseText);
            if (data == null || typeof (data) == 'undefined') {
                data = $.parseJSON(data.firstChild.textContent);
            }
            //success
        };
        xdr.onerror = function (e) {
            //error
        }
        xdr.open("GET", url);
        xdr.send();
```

5.问题算是解决了get都可以了。还有问题IE11你又出来干嘛了。。。你居然没有XDomainRequest。多么坑爹啊。还好IE11可以用标准的提交跨域。只是判别的时候有点蛋疼。

```javascript

navigator.userAgent.toLowerCase().match(/(msie\s|trident.*rv:)([\w.]+)/)

```

这个能否判断IE11自己试着办哈。。


***本人水平有限，如果有所遗漏和谬误，请各位朋友指正，希望一起讨论学习和进步***