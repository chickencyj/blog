---
layout: post
title: v8的handles,scopes和contexts
date: 2017-3-26
categories: blog
tags: [V8]
description: handles,scopes和contexts
---

### 关于引用的疑问？
从学习js以来，就知道ECMAScript变量包含两种不同类型的值：基本类型值、引用类型值。基本类型保存在栈中，引用类型保存在堆中。基本类型可以按值访问，而引用类型则是按引用访问。而按引用访问是类似指针的方式在栈中先找到内存地址，再去读取保存在堆中的值。但对引用的了解仅限于此，一直不明白具体的引用指的是什么。这几天翻了一下v8的wiki了解到了引用具体指什么。

### Handles and Handle Scope
handle 是什么？从广义上，能够从一个数值拎起一大堆数据的东西都可以叫做handle。在v8中，Handle提供了一个JS对象在堆内存中的地址的引用，而v8中的GC则通过管理handle的方式去跟踪管理JS对象。当一个对象已经无法从根节点访问到，GC将会回收它。而在回收过程中，GC会将对象在堆内存中进行移动。同时也会将所有相应的handle更新为新的地址。

* Local Handles 保存在栈中，其生命周期取决于handle scope，当一个handle scope被销毁时,当中所有的handle也被销毁，而这些handle所引用的的对象将在下次GC的时候将被进行适当的处理。

* Persistent handle 不受handle Scope 管理，类似于全局，可以延伸到不同函数

* Eternal 是一个用于预期永远不会被释放的javaScript对象的persistent handle, 使用它的代价更小, 因为它减轻了垃圾回收器判定对象是否存活的负担.

### Contexts
在 V8 中,context使得可以在一个V8实例中运行相互隔离且无关的javaScript代码,javaScript代码必须运行在context中.当你创建一个 context 后, 你可以进出此上下文任意多的次数. 当你在 context A 中时, 还可以再进入 context B. 此时你将进入 B 的上下文中. 当退出 B 时, A 又将成为你的当前 context.

```sh
 // Create a new context.
 Local<Context> context = Context::New(isolate);
 // Enter the context for compiling and running the hello world script.
 Context::Scope context_scope(context);
 // Create a string containing the JavaScript source code.
 Local<String> source =
 String::NewFromUtf8(isolate, "'Hello' + ', World!'",
 NewStringType::kNormal).ToLocalChecked();
 // Compile the source code.
 Local<Script> script = Script::Compile(context, source).ToLocalChecked();
```
