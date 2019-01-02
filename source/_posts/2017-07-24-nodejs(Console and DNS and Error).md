---
layout: post
title: Console DNS and Error
date: 2017-7-24
categories: blog
tags: [node]
description: nodejs Console DNS and Error
---


刚刚又看了下Console, DNS 和 Error, 遇到一些有趣的知识点，总结一下

### Console
* nodejs默认情况下我们的console的输出会发送到 process.stdout 和 process.stderr，其实就是调用
```
new Console(process.stdout, process.stderr)
```
* new Console 接受两个两个可写流参数(stdout <Writable>， stderr <Writable>)，cosnole.log输出到stdout，console.error输出到stderr。我们可以自定义console，通过传入不同的流把console写到不同的地方
* console 是异步还是同步，根据不同的操作系统以及传入的不同的可写流参数来区分
 * Files: 在Windows and Linux 都是同步的
 * TTYs (Terminals): Windows 异步, Unix 同步
 * Pipes (and sockets): Windows 同步, Unix 

### DNS
* DNS模块有两种函数都是用来域名解析 dns.lookup('xxxx', callback) [使用底层操作系统工具进行域名解析] 以及 dns.resolve4('xxx', callback) 或者 dns.resolve('xxx', 'A', callback) [连接到一个真实的 DNS 服务器进行域名解析]
*  dns.lookup 会同步的调用getaddrinfo(3),这可能会阻塞libuv线程池中其它的操作， 而dns.resolve通过网络执行异步IO来域名解析，没有阻塞其它操作的风险

### Error
* JavaScript 的 throw 机制的任何使用都会引起异常，异常必须使用 try / catch 处理，否则 Node.js 进程会立即退出。
* 而try / catch 不能用于捕获由异步 API 引起的错误，因为当回调被调用时，try/catch 早已执行完被回收了。这种情况我们可以用process.on('uncaughtException'， callback)来监听和在callback处理没有被捕捉到的异常

