---
layout: post
title:  前端小技巧总结
category : 前端技术
tags : 前端 js
---
===


### 生成随机字符串

利用`Math.random`和`toString`生成随机字符串，来自前一阵子看到的一篇博文。这里的技巧是利用了`toString`方法可以接收一个基数作为参数的原理，这个基数从2到36封顶。如果不指定，默认基数是10进制。略屌！    


```javascript
function generateRandomAlphaNum(len) {
    var rdmString = "";
    for (; rdmString.length < len; rdmString += Math.random().toString(36).substr(2));
    return rdmString.substr(0, len);
}
```

### 整数的操作

JavaScript中是没有整型概念的，但利用好位操作符可以轻松处理，同时获得效率上的提升。

`|0和~~`是很好的一个例子，使用这两者可以将浮点转成整型且效率方面要比同类的parseInt,Math.round 要快。在处理像素及动画位移等效果的时候会很有用。性能比较见此。

```javascript
var foo = (12.4 / 4.13) | 0;//结果为3
var bar = ~~(12.4 / 4.13);//结果为3
```

 顺便说句，`!!`将一个值方便快速转化为布尔值 `!!window===true` 。





### If语句的变形

当你需要写一个if语句的时候，不妨尝试另一种更简便的方法，用JavaScript中的逻辑操作符来代替。

```javascript
var day=(new Date).getDay()===0;
//传统if语句
if (day) {
	alert('Today is Sunday!');
};
//运用逻辑与代替if
day&&alert('Today is Sunday!');
```

比如上面的代码，首先得到今天的日期，如果是星期天，则弹窗，否则什么也不做。我们知道逻辑操作存在短路的情况，对于逻辑与表达式，只有两者都真才结果才为真，如果前面的day变量被判断为假了，那么对于整个与表达式来说结果就是假，所以就不会继续去执行后面的alert了，如果前面day为真，则还要继续执行后面的代码来确定整个表达式的真假。利用这点达到了if的效果。



### 禁止别人以iframe加载你的页面

下面的代码已经不言自明了，没什么好多说的。

```javascript
if (window.location != window.parent.location) window.parent.location = window.location;

console.table
```

### Chrome专属，IE绕道的console方法

可以将JavaScript关联数组以表格形式输出到浏览器console，效果很惊赞，界面很美观。

```javascript
//采购情况
var data = [{'品名': '杜雷斯', '数量': 4}, {'品名': '冈本', '数量': 3}];
console.table(data);
```

### 浏览器小技巧

Chrome Canary 开发者工具可以控制 CSS3 动画的暂停以及速度啦！

Chrome 打开 DevTools 之后，右击或者一直摁住刷新键，会出来清缓存强制刷新的功能选项( tip from Addy Osmani ) ，其中「硬性重新加载」类似于强制刷新。

 Chrome DevTools 的 Network 面板现在支持使用「-」反向搜索了。例如输入「-.js」会筛选出所有非 JS 的请求。
 
在 Chrome 和 Safari 的开发中工具中，可以按「H」键快速隐藏元素。

chrome开发工具技巧：用放大镜选中了某个DOM之后，在console中输入$0，就得到当前DOM的引用，临时调试起来就方便很多了！
