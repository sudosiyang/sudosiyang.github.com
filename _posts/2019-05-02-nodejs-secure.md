---
layout: post
title: Nodejs-应用安全清单
category : 前端技术
tags : 前端 JavaScript nodejs
---

## helmet 设置安全响应头

检测头部配置：[Security Headers](https://securityheaders.com/)。

应用程序应该使用安全的 header 来防止攻击者使用常见的攻击方式，诸如跨站点脚本攻击(XSS)、跨站请求伪造(CSRF)。可以使用模块 [helmet](https://www.npmjs.com/package/helmet) 轻松进行配置。

- 构造
    - X-Frame-Options：sameorigin。提供点击劫持保护，iframe 只能同源。
- 传输
    - Strict-Transport-Security：max-age=31536000; includeSubDomains。强制 HTTPS，这减少了web 应用程序中错误通过 cookies 和外部链接，泄露会话数据，并防止中间人攻击
- 内容
    - X-Content-Type-Options：nosniff。阻止从声明的内容类型中嗅探响应，减少了用户上传恶意内容造成的风险
    - Content-Type：text/html;charset=utf-8。指示浏览器将页面解释为特定的内容类型，而不是依赖浏览器进行假设
- XSS
    - X-XSS-Protection：1; mode=block。启用了内置于最新 web 浏览器中的跨站点脚本(XSS)过滤器
- 下载
    - X-Download-Options：noopen。
- 缓存
    - Cache-Control：no-cache。web 应中返回的数据可以由用户浏览器以及中间代理缓存。该指令指示他们不要保留页面内容，以免其他人从这些缓存中访问敏感内容
    - Pragma：no-cache。同上
    - Expires：-1。web 响应中返回的数据可以由用户浏览器以及中间代理缓存。该指令通过将到期时间设置为一个值来防止这种情况。
- 访问控制
    - Access-Control-Allow-Origin：not *。'Access-Control-Allow-Origin: *' 默认在现代浏览器中禁用
    - X-Permitted-Cross-Domain-Policies：master-only。指示只有指定的文件在此域中才被视为有效
- 内容安全策略
    - Content-Security-Policy：内容安全策略需要仔细调整并精确定义策略
- 服务器信息
    - Server：不显示。

## 使用 security-linter 插件

使用安全检验插件 [eslint-plugin-security](https://github.com/nodesecurity/eslint-plugin-security) 或者 [tslint-config-security](https://www.npmjs.com/package/tslint-config-security)。

## koa-ratelimit 限制并发请求

DOS 攻击非常流行而且相对容易处理。使用外部服务，比如 cloud 负载均衡, cloud 防火墙, nginx, 或者（对于小的，不是那么重要的app）一个速率限制中间件(比如 [koa-ratelimit](https://github.com/koajs/ratelimit))，来实现速率限制。

## 纯文本机密信息放置

存储在源代码管理中的机密信息必须进行加密和管理 (滚动密钥(rolling keys)、过期时间、审核等)。使用 pre-commit/push 钩子防止意外提交机密信息。

## ORM/ODM 库防止查询注入漏洞

要防止 SQL/NoSQL 注入和其他恶意攻击, 请始终使用 ORM/ODM 或 database 库来转义数据或支持命名的或索引的参数化查询, 并注意验证用户输入的预期类型。不要只使用 JavaScript 模板字符串或字符串串联将值插入到查询语句中, 因为这会将应用程序置于广泛的漏洞中。

库：

- TypeORM
- sequelize
- mongoose
- Knex
- Objection.js
- waterline

## 使用 Bcrypt 代替 Crypto

密码或机密信息(API 密钥)应该使用安全的 hash + salt 函数([bcrypt](https://www.npmjs.com/package/bcrypt))来存储, 因为性能和安全原因, 这应该是其 JavaScript 实现的首选。

```js
// 使用10个哈希回合异步生成安全密码
bcrypt.hash('myPassword', 10, function(err, hash) {
  // 在用户记录中存储安全哈希
});

// 将提供的密码输入与已保存的哈希进行比较
bcrypt.compare('somePassword', hash, function(err, match) {
  if(match) {
   // 密码匹配
  } else {
   // 密码不匹配
  } 
});
```

## 转义 HTML、JS 和 CSS 输出

发送给浏览器的不受信任数据可能会被执行, 而不是显示, 这通常被称为跨站点脚本(XSS)攻击。使用专用库将数据显式标记为不应执行的纯文本内容(例如:编码、转义)，可以减轻这种问题。

## 验证传入的 JSON schemas

验证传入请求的 body payload，并确保其符合预期要求, 如果没有, 则快速报错。为了避免每个路由中繁琐的验证编码, 您可以使用基于 JSON 的轻量级验证架构，比如 [jsonschema](https://www.npmjs.com/package/jsonschema) 或 [joi](https://www.npmjs.com/package/joi)

## 支持黑名单的 JWT

当使用 JSON Web Tokens(例如, 通过 [Passport.js](https://github.com/jaredhanson/passport)), 默认情况下, 没有任何机制可以从发出的令牌中撤消访问权限。一旦发现了一些恶意用户活动, 只要它们持有有效的标记, 就无法阻止他们访问系统。通过实现一个不受信任令牌的黑名单，并在每个请求上验证，来减轻此问题。

```js
const jwt = require('express-jwt');
const blacklist = require('express-jwt-blacklist');
 
app.use(jwt({
  secret: 'my-secret',
  isRevoked: blacklist.isRevoked
}));
 
app.get('/logout', function (req, res) {
  blacklist.revoke(req.user)
  res.sendStatus(200);
});
```

## 限制每个用户允许的登录请求

一类保护暴力破解的中间件，比如 express-brute，应该被用在 express 的应用中，来防止暴力/字典攻击；这类攻击主要应用于一些敏感路由，比如 `/admin` 或者 `/login`，基于某些请求属性, 如用户名, 或其他标识符, 如正文参数等。否则攻击者可以发出无限制的密码匹配尝试, 以获取对应用程序中特权帐户的访问权限。

```js
const ExpressBrute = require('express-brute');
const RedisStore = require('express-brute-redis');

const redisStore = new RedisStore({
  host: '127.0.0.1',
  port: 6379
});

// Start slowing requests after 5 failed 
// attempts to login for the same user
const loginBruteforce = new ExpressBrute(redisStore, {
  freeRetries: 5,
  minWait: 5 * 60 * 1000, // 5 minutes
  maxWait: 60 * 60 * 1000, // 1 hour
  failCallback: failCallback,
  handleStoreError: handleStoreErrorCallback
});

app.post('/login',
  loginBruteforce.getMiddleware({
    key: function (req, res, next) {
      // prevent too many attempts for the same username
      next(req.body.username);
    }
  }), // error 403 if we hit this route too often
  function (req, res, next) {
    if (User.isValidLogin(req.body.username, req.body.password)) {
      // reset the failure counter for valid login
      req.brute.reset(function () {
        res.redirect('/'); // logged in
      });
    } else {
      // handle invalid user
    }
  }
);
```

## 使用非 root 用户运行 Node.js

Node.js 作为一个具有无限权限的 root 用户运行，这是一种普遍的情景。例如，在 Docker 容器中，这是默认行为。建议创建一个非 root 用户，并保存到 Docker 镜像中（下面给出了示例），或者通过调用带有"-u username" 的容器来代表此用户运行该进程。否则在服务器上运行脚本的攻击者在本地计算机上获得无限制的权利 (例如，改变 iptable，引流到他的服务器上)

```dockerfile
FROM node:latest
COPY package.json .
RUN npm install
COPY . .
EXPOSE 3000
USER node
CMD ["node", "server.js"]
```

## 使用反向代理或中间件限制负载大小

请求 body 有效载荷越大, Node.js 的单线程就越难处理它。这是攻击者在没有大量请求(DOS/DDOS 攻击)的情况下，就可以让服务器跪下的机会。在边缘上（例如，防火墙，ELB）限制传入请求的 body 大小，或者通过配置 `express body parser` 仅接收小的载荷，可以减轻这种问题。否则您的应用程序将不得不处理大的请求, 无法处理它必须完成的其他重要工作, 从而导致对 DOS 攻击的性能影响和脆弱性。

express：

```js
const express = require('express');

const app = express();

// body-parser defaults to a body size limit of 300kb
app.use(express.json({ limit: '300kb' })); 

// Request with json body
app.post('/json', (req, res) => {

    // Check if request payload content-type matches json
    // because body-parser does not check for content types
    if (!req.is('json')) {
        return res.sendStatus(415); // Unsupported media type if request doesn't have JSON body
    }

    res.send('Hooray, it worked!');
});

app.listen(3000, () => console.log('Example app listening on port 3000!'));
```

nginx：

```
http {
    ...
    # Limit the body size for ALL incoming requests to 1 MB
    client_max_body_size 1m;
}

server {
    ...
    # Limit the body size for incoming requests to this specific server block to 1 MB
    client_max_body_size 1m;
}

location /upload {
    ...
    # Limit the body size for incoming requests to this route to 1 MB
    client_max_body_size 1m;
}
```

## 防止 RegEx 让 NodeJS 过载

匹配文本的用户输入需要大量的 CPU 周期来处理。在某种程度上，正则处理是效率低下的，比如验证 10 个单词的单个请求可能阻止整个 event loop 长达6秒。由于这个原因，偏向第三方的验证包，比如[validator.js](https://github.com/chriso/validator.js)，而不是采用正则，或者使用 [safe-regex](https://github.com/substack/safe-regex) 来检测有问题的正则表达式。

```js
const saferegex = require('safe-regex');
const emailRegex = /^([a-zA-Z0-9])(([\-.]|[_]+)?([a-zA-Z0-9]+))*(@){1}[a-z0-9]+[.]{1}(([a-z]{2,3})|([a-z]{2,3}[.]{1}[a-z]{2,3}))$/;

// should output false because the emailRegex is vulnerable to redos attacks
console.log(saferegex(emailRegex));

// instead of the regex pattern, use validator:
const validator = require('validator');
console.log(validator.isEmail('liran.tal@gmail.com'));
```

## 在沙箱中运行不安全代码

当任务执行在运行时给出的外部代码时(例如, 插件), 使用任何类型的沙盒执行环境保护主代码，并隔离开主代码和插件。这可以通过一个专用的过程来实现 (例如:cluster.fork()), 无服务器环境或充当沙盒的专用 npm 包。

- 一个专门的子进程 - 这提供了一个快速的信息隔离, 但要求制约子进程, 限制其执行时间, 并从错误中恢复
- 一个基于云的无服务框架满足所有沙盒要求，但动态部署和调用Faas方法不是本部分的内容
- 一些 npm 库，比如 [sandbox](https://www.npmjs.com/package/sandbox) 和 [vm2](https://www.npmjs.com/package/vm2) 允许通过一行代码执行隔离代码。尽管后一种选择在简单中获胜, 但它提供了有限的保护。

```js
const Sandbox = require("sandbox");
const s = new Sandbox();

s.run( "lol)hai", function( output ) {
  console.log(output);
  //output='Synatx error'
});

// Example 4 - Restricted code
s.run( "process.platform", function( output ) {
  console.log(output);
  //output=Null
})

// Example 5 - Infinite loop
s.run( "while (true) {}", function( output ) {
  console.log(output);
  //output='Timeout'
})
```

## 隐藏客户端的错误详细信息

默认情况下, 集成的 express 错误处理程序隐藏错误详细信息。但是, 极有可能, 您实现自己的错误处理逻辑与自定义错误对象(被许多人认为是最佳做法)。如果这样做, 请确保不将整个 Error 对象返回到客户端, 这可能包含一些敏感的应用程序详细信息。否则敏感应用程序详细信息(如服务器文件路径、使用中的第三方模块和可能被攻击者利用的应用程序的其他内部工作流)可能会从 stack trace 发现的信息中泄露。

```
// production error handler
// no stacktraces leaked to user
app.use(function(err, req, res, next) {
    res.status(err.status || 500);
    res.render('error', {
        message: err.message,
        error: {}
    });
});
```

## 对 npm 或 Yarn，配置 2FA

开发链中的任何步骤都应使用 MFA(多重身份验证)进行保护, npm/Yarn 对于那些能够掌握某些开发人员密码的攻击者来说是一个很好的机会。使用开发人员凭据, 攻击者可以向跨项目和服务广泛安装的库中注入恶意代码。甚至可能在网络上公开发布。在 npm 中启用两层身份验证（2-factor-authentication）, 攻击者几乎没有机会改变您的软件包代码。

https://itnext.io/eslint-backdoor-what-it-is-and-how-to-fix-the-issue-221f58f1a8c8

## session 中间件设置

每个 web 框架和技术都有其已知的弱点，告诉攻击者我们使用的 web 框架对他们来说是很大的帮助。使用 session 中间件的默认设置, 可以以类似于 `X-Powered-Byheader` 的方式向模块和框架特定的劫持攻击公开您的应用。尝试隐藏识别和揭露技术栈的任何内容(例如:Nonde.js, express)。否则可以通过不安全的连接发送cookie, 攻击者可能会使用会话标识来标识web应用程序的基础框架以及特定于模块的漏洞。

```js
// using the express session middleware
app.use(session({  
 secret: 'youruniquesecret', // secret string used in the signing of the session ID that is stored in the cookie
 name: 'youruniquename', // set a unique name to remove the default connect.sid
 cookie: {
   httpOnly: true, // minimize risk of XSS attacks by restricting the client from reading the cookie
   secure: true, // only send cookie over https
   maxAge: 60000*60*24 // set cookie expiry length in ms
 }
}));
```

## csurf 防止 CSRF

路由层：

```js
var cookieParser = require('cookie-parser');  
var csrf = require('csurf');  
var bodyParser = require('body-parser');  
var express = require('express');

// 设置路由中间件
var csrfProtection = csrf({ cookie: true });  
var parseForm = bodyParser.urlencoded({ extended: false });

var app = express();

// 我们需要这个，因为在 csrfProtection 中 “cookie” 是正确的
app.use(cookieParser());

app.get('/form', csrfProtection, function(req, res) {  
  // 将 CSRFToken 传递给视图
  res.render('send', { csrfToken: req.csrfToken() });
});

app.post('/process', parseForm, csrfProtection, function(req, res) {  
  res.send('data is being processed');
});
```

展示层：

```html
<form action="/process" method="POST">  
  <input type="hidden" name="_csrf" value="{{csrfToken}}">

  Favorite color: <input type="text" name="favoriteColor">
  <button type="submit">Submit</button>
</form>  
```
