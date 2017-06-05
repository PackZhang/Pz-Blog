---
title: ReactiveCocoa
date: 2017-05-12 16:57:20
tags: 
- ReactiveCocoa
---
## 写在前面
> ReactiveCocoa（其简称为 RAC）是由 Github 开源的一个应用于 iOS 和 OS X 开发的新框架。RAC 具有函数式编程和响应式编程的特性。它主要吸取了 .Net 的 Reactive Extensions 的设计和实现。
但是，相对于传统的 MVC 架构，ReactiveCocoa 的函数式编程方式的学习曲线陡峭，业界也没有丰富的图书资料，这使得大家对这种技术望而却步。

以上引用自[唐巧的博客](http://blog.devtang.com/2016/01/03/reactive-cocoa-discussion/)。从自己学习的情况来看，学习ReactiveCocoa的曲线还可以接受，远没到望而却步的地步。先学习常规的用法，清楚`RACSignal`，`RACCommand`，`RACSubject`这三中信号的原理，用法，场景。

再去尝试使用MVVM+RAC的结构去构建你的应用。

## RAC常用类的继承关系
<center>![](http://i2.muimg.com/1949/2b3967b71cb5d0d0.png)</center>

##常用类
[ReactiveCocoa之核心信号---RACSignal](http://104.224.135.25/2017/05/17/ReactiveCocoa-RACSignal/)
[ReactiveCocoa之核心信号---RACCommand](http://104.224.135.25/2017/05/31/ReactiveCocoa-RACCommand/)