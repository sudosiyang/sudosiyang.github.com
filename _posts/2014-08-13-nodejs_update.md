---
layout: post
title:  node.js升级
category : 前端技术
tags : 前端 node
---
===
今天，又发现一个超级简单的升级node.js的方法。一行命令搞定，省去了重新编译安装的过程。

node有一个模块叫n（这名字可够短的。。。），是专门用来管理node.js的版本的。
首先安装n模块：
```

npm install -g n

```

第二步：

升级node.js到最新稳定版

```

n stable

```

是不是很简单？！

n后面也可以跟随版本号比如：

```

n v0.10.29

```

或

```

n 0.10.29

```
