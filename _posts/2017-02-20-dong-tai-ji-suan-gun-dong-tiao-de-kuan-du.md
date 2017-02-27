---
layout: post
title: 动态计算滚动条的宽度
category : 前端技术
tags : 前端 javascript
---

其实，很多情况下滚动条宽度是不用计算的，特别是谷歌浏览器，可以对滚动条进行美化。

在PC浏览器中，滚动条是占位元素的内边距和内容区域的；而在移动浏览器中，滚动条是不占用内边距和内容区域，并且还及时显隐。因此，只需要在在PC浏览器中计算滚动条的宽度，尤其是在全屏弹窗不可滚动的情况中。


那么如何计算滚动条的宽度呢，其实很简单，就是创建一个没有滚动条的div获取他的宽度，在创建一个有滚动条的div取其宽。然后做个简单的减法就可以了。

代码如下

```js
function scrollBarWidth() {
  const outer = document.createElement('div');
  outer.className = 'scrollbar__wrap';
  outer.style.visibility = 'hidden';
  outer.style.width = '100px';
  outer.style.position = 'absolute';
  outer.style.top = '-9999px';
  document.body.appendChild(outer);

  const widthNoScroll = outer.offsetWidth;
  outer.style.overflow = 'scroll';

  const inner = document.createElement('div');
  inner.style.width = '100%';
  outer.appendChild(inner);

  const widthWithScroll = inner.offsetWidth;
  outer.parentNode.removeChild(outer);

  return widthNoScroll - widthWithScroll;
};
```

只是介绍了一个计算滚动条的方法，大家在用到的时候可以参考一下，希望对您有帮助！