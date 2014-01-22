---
layout: post
title:  JavaScript 的HTML转义方法
category : 前端技术
tags : 前端 P3P
---
===
此方法用来将用户输入内容中的尖括号、引号等进行转义

正常版，可在nodejs中使用

```javascript

function html_encode(str) {
    var s = "";
    if (str.length == 0) return "";
    s = str.replace(/&/g, "&gt;");
    s = s.replace(/</g, "&lt;");
    s = s.replace(/>/g, "&gt;");
    s = s.replace(/ /g, "&nbsp;");
    s = s.replace(/\'/g, "&#39;");
    s = s.replace(/\"/g, "&quot;");
    s = s.replace(/\n/g, "<br>");
    return s;
}

function html_decode(str) {
    var s = "";
    if (str.length == 0) return "";
    s = str.replace(/&gt;/g, "&");
    s = s.replace(/&lt;/g, "<");
    s = s.replace(/&gt;/g, ">");
    s = s.replace(/&nbsp;/g, " ");
    s = s.replace(/&#39;/g, "\'");
    s = s.replace(/&quot;/g, "\"");
    s = s.replace(/<br>/g, "\n");
    return s;
}

```

巧妙版：


```javascript

function htmlEncode(str) {

    var div = document.createElement("div");

    div.appendChild(document.createTextNode(str));

    return div.innerHTML;

}

function htmlDecode(str) {

    var div = document.createElement("div");

    div.innerHTML = str;

    return div.innerHTML;

}


```
