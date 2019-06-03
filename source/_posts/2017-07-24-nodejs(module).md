---
layout: post
title: module
date: 2017-7-24
categories: [node, commonjs]
tags: [node]
description: nodejs module
---
今天又看了一下node的module，以及其载入策略，记一下笔记不要忘了(以前看过然后看完就忘。。。也是醉了)

### module基础概念
* 在node中，模块与文件是一一对应的，一个文件就是一个module，通过把属性或方法挂载到exports，在其他模块中可以通过require获取这些方法或属性。
* 模块内的本地变量是私有的，为啥？因为对于每个模块node都会把它包装在一个函数中,如此便可以避免作用域的污染同时提供一些看似全局实际是每个模块特定的变量
```
(function(exports, require, module, __filename, __dirname) {
// 模块的代码实际上在这里
})
```
* 循环调用 require() 时， 为了防止无限的循环，会返回一个模块 的 exports 对象的 未完成的副本 给 另一个 模块，具体得看代码怎么写，官网有很详细的例子

### exports 与 module.exports
* 简单的说，exports是module.exports的引用，而require获得的返回值是module.exports,因此当exports = {},这样就改变了它的引用，从而不再指向module.exports.
```
function require(/* ... */) {
  const module = { exports: {} };
  ((module, exports) => {
    // 模块代码在这。在这个例子中，定义了一个函数。
    function someFunc() {}
    exports = someFunc;
    // 此时，exports 不再是一个 module.exports 的快捷方式，
    // 且这个模块依然导出一个空的默认对象。
    module.exports = someFunc;
    // 此时，该模块导出 someFunc，而不是默认对象。
  })(module, module.exports);
  return module.exports;
}
```

### 模块分类
* Node.js的模块分为两类，一类为原生（核心）模块，一类为文件模块。
文件模块中，又分为3类模块
* js。通过fs模块同步读取js文件并编译执行。
* node。通过C/C++进行编写的Addon。通过dlopen方法进行加载作为二进制插件。
* json。读取文件，调用JSON.parse解析成一个 JavaScript 对象加载。

### 模块加载
* 是否在文件模块缓存区中
* 是否原生模块 => 否 => 查找文件模块
* 是否原生模块 => 是 => 是否在原生模块缓存区中 => 否 => 加载原生模块
