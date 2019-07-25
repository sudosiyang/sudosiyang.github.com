---
layout: post
title: Nodejs-多进程、多线程和分布式
category : 前端技术
tags : 前端 JavaScript nodejs
---

# 子进程

## 执行外部应用

### 基本概念

- 4个异步方法：exec、execFile、fork、spawn
    - Node
        - fork：想将一个 Node 进程作为一个独立的进程来运行的时候使用，是的计算处理和文件描述器脱离 Node 主进程
    - 非 Node
        - spawn：处理一些会有很多子进程 I/O 时、进程会有大量输出时使用
        - execFile：只需执行一个外部程序的时候使用，执行速度快，处理用户输入相对安全
        - exec：想直接访问线程的 shell 命令时使用，一定要注意用户输入
- 3个同步方法：execSync、execFileSync、spawnSync
- 通过 API 创建出来的子进程和父进程没有任何必然联系

### execFile

- 会把输出结果缓存好，通过回调返回最后结果或者异常信息

```js
const cp = require('child_process');

cp.execFile('echo', ['hello', 'world'], (err, stdout, stderr) => {
  if (err) { console.error(err); }
  console.log('stdout: ', stdout);
  console.log('stderr: ', stderr);
});
```

### spawn

- 通过流可以使用有大量数据输出的外部应用，节约内存
- 使用流提高数据响应效率
- spawn 方法返回一个 I/O 的流接口

#### 单一任务

```js
const cp = require('child_process');

const child = cp.spawn('echo', ['hello', 'world']);
child.on('error', console.error);
child.stdout.pipe(process.stdout);
child.stderr.pipe(process.stderr);
```

#### 多任务串联

```js
const cp = require('child_process');
const path = require('path');

const cat = cp.spawn('cat', [path.resolve(__dirname, 'messy.txt')]);
const sort = cp.spawn('sort');
const uniq = cp.spawn('uniq');

cat.stdout.pipe(sort.stdin);
sort.stdout.pipe(uniq.stdin);
uniq.stdout.pipe(process.stdout);
```

### exec

- 只有一个字符串命令
- 和 shell 一模一样

```js
const cp = require('child_process');

cp.exec(`cat ${__dirname}/messy.txt | sort | uniq`, (err, stdout, stderr) => {
  console.log(stdout);
});
```

### fork

- fork 方法会开发一个 IPC 通道，不同的 Node 进程进行消息传送
- 一个子进程消耗 30ms 启动时间和 10MB 内存
- 子进程：`process.on('message')`、`process.send()`
- 父进程：`child.on('message')`、`child.send()`

#### 父子通信

```js
// parent.js
const cp = require('child_process');

const child = cp.fork('./child', { silent: true });
child.send('monkeys');
child.on('message', function (message) {
  console.log('got message from child', message, typeof message);
})
child.stdout.pipe(process.stdout);

setTimeout(function () {
  child.disconnect();
}, 3000);
```

```js
// child.js
process.on('message', function (message) {
  console.log('got one', message);
  process.send('no pizza');
  process.send(1);
  process.send({ my: 'object' });
  process.send(false);
  process.send(null);
});

console.log(process);
```

## 常用技巧

### 退出时杀死所有子进程

- 保留对由 spawn 返回的 ChildProcess 对象的引用，并在退出主进程时将其杀死

```js
const spawn = require('child_process').spawn;
const children = [];

process.on('exit', function () {
  console.log('killing', children.length, 'child processes');
  children.forEach(function (child) {
    child.kill();
  });
});

children.push(spawn('/bin/sleep', ['10']));
children.push(spawn('/bin/sleep', ['10']));
children.push(spawn('/bin/sleep', ['10']));

setTimeout(function () { process.exit(0); }, 3000);
```

## Cluster 的理解

- 解决 NodeJS 单进程无法充分利用多核 CPU 问题
- 通过 master-cluster 模式可以使得应用更加健壮
- Cluster 底层是 child_process 模块，除了可以发送普通消息，还可以发送底层对象 `TCP`、`UDP` 等
- TCP 主进程发送到子进程，子进程能根据消息重建出 TCP 连接，Cluster 可以决定 fork 出合适的硬件资源的子进程数

# Node 多线程

## 单线程问题

- 对 cpu 利用不足
- 某个未捕获的异常可能会导致整个程序的退出

## Node 线程

- Node 进程占用了 7 个线程
- Node 中最核心的是 v8 引擎，在 Node 启动后，会创建 v8 的实例，这个实例是多线程的
    - 主线程：编译、执行代码
    - 编译/优化线程：在主线程执行的时候，可以优化代码
    - 分析器线程：记录分析代码运行时间，为 Crankshaft 优化代码执行提供依据
    - 垃圾回收的几个线程
- JavaScript 的执行是单线程的，但 Javascript 的宿主环境，无论是 Node 还是浏览器都是多线程的

## 异步 IO

- Node 中有一些 IO 操作（DNS，FS）和一些 CPU 密集计算（Zlib，Crypto）会启用 Node 的线程池
- 线程池默认大小为 4，可以手动更改线程池默认大小

```js
process.env.UV_THREADPOOL_SIZE = 64
```

## cluster 多进程

```js
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log(`主进程 ${process.pid} 正在运行`);
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`工作进程 ${worker.process.pid} 已退出`);
  });
} else {
  // 工作进程可以共享任何 TCP 连接。
  // 在本例子中，共享的是 HTTP 服务器。
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('Hello World');
  }).listen(8000);
  console.log(`工作进程 ${process.pid} 已启动`);
}
```

- 一共有 9 个进程，其中一个主进程，cpu 个数 x cpu 核数 = 2 x 4 = 8 个 子进程
- 无论 child_process 还是 cluster，都不是多线程模型，而是多进程模型
- 应对单线程问题，通常使用多进程的方式来模拟多线程

## 真 Node 多线程

- Node 10.5.0 的发布，给出了一个实验性质的模块 worker_threads 给 Node 提供真正的多线程能力
- worker_thread 模块中有 4 个对象和 2 个类
    - isMainThread: 是否是主线程，源码中是通过 threadId === 0 进行判断的。
    - MessagePort: 用于线程之间的通信，继承自 EventEmitter。
    - MessageChannel: 用于创建异步、双向通信的通道实例。
    - threadId: 线程 ID。
    - Worker: 用于在主线程中创建子线程。第一个参数为 filename，表示子线程执行的入口。
    - parentPort: 在 worker 线程里是表示父进程的 MessagePort 类型的对象，在主线程里为 null
    - workerData: 用于在主进程中向子进程传递数据（data 副本）

```js
const {
  isMainThread,
  parentPort,
  workerData,
  threadId,
  MessageChannel,
  MessagePort,
  Worker
} = require('worker_threads');

function mainThread() {
  for (let i = 0; i < 5; i++) {
    const worker = new Worker(__filename, { workerData: i });
    worker.on('exit', code => { console.log(`main: worker stopped with exit code ${code}`); });
    worker.on('message', msg => {
      console.log(`main: receive ${msg}`);
      worker.postMessage(msg + 1);
    });
  }
}

function workerThread() {
  console.log(`worker: workerDate ${workerData}`);
  parentPort.on('message', msg => {
    console.log(`worker: receive ${msg}`);
  }),
  parentPort.postMessage(workerData);
}

if (isMainThread) {
  mainThread();
} else {
  workerThread();
}
```

### 线程通信

```js
const assert = require('assert');
const {
  Worker,
  MessageChannel,
  MessagePort,
  isMainThread,
  parentPort
} = require('worker_threads');
if (isMainThread) {
  const worker = new Worker(__filename);
  const subChannel = new MessageChannel();
  worker.postMessage({ hereIsYourPort: subChannel.port1 }, [subChannel.port1]);
  subChannel.port2.on('message', (value) => {
    console.log('received:', value);
  });
} else {
  parentPort.once('message', (value) => {
    assert(value.hereIsYourPort instanceof MessagePort);
    value.hereIsYourPort.postMessage('the worker is sending this');
    value.hereIsYourPort.close();
  });
}
```

## 多进程 vs 多线程

进程是资源分配的最小单位，线程是CPU调度的最小单位

# 分布式

## 分布式锁

- 在单机场景下，可以使用语言的内置锁来实现进程同步
- 在分布式场景下，需要同步的进程可能位于不同的节点上，那么就需要使用分布式锁

### 数据库的唯一索引

获得锁时向表中插入一条记录，释放锁时删除这条记录。

- 锁没有失效时间，解锁失败的话其它进程无法再获得该锁。
- 只能是非阻塞锁，插入失败直接就报错了，无法重试。
- 不可重入，已经获得锁的进程也必须重新获取锁。

### Redis 的 SETNX 指令

使用 SETNX（set if not exist）指令插入一个键值对，如果 Key 已经存在，那么会返回 False，否则插入成功并返回 True

- SETNX 指令和数据库的唯一索引类似，保证了只存在一个 Key 的键值对，可以用一个 Key 的键值对是否存在来判断是否存于锁定状态
- EXPIRE 指令可以为一个键值对设置一个过期时间，从而避免了数据库唯一索引实现方式中释放锁失败的问题

### Redis 的 RedLock 算法

使用了多个 Redis 实例来实现分布式锁，这是为了保证在发生单点故障时仍然可用

- 尝试从 N 个相互独立 Redis 实例获取锁
- 计算获取锁消耗的时间，只有当这个时间小于锁的过期时间，并且从大多数（N / 2 + 1）实例上获取了锁，那么就认为锁获取成功了
- 如果锁获取失败，就到每个实例上释放锁