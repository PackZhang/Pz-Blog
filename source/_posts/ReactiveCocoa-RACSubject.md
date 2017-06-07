---
title: ReactiveCocoa之核心信号---RACSubject
date: 2017-06-07 14:04:28
tags: ReactiveCocoa
---
## 代替代理
`RACSubject`本身可以当做信号，也可以订阅信号。`RACSubject`常常可以用来替代代理，使用方式简单。
![](http://i1.piimg.com/1949/04ae35a4adfec08e.jpg)

原来实现代理需要六步，实现起来比较麻烦，如果使用RAC三步就可以达到代理的效果。

### 创建信号
```
RACSubject *subject = [RACSubject subject];
```
RAC提供了工厂方法可以快速创建一个`RACSubject`对象。与`RACSignal`不同的是不需要在创建时就去添加`didSubscribe`闭包，也不需要在`didSubscribe `闭包中去调用`sendNext`方法。
### 订阅信号
```
[subject subscribeNext:^(id  _Nullable x) {
	//do something
}];
```
`nextBlock`闭包在执行`sendNext`方法时触发。
### 发送信号
```
[subject sendNext:@3];
```
***
在实际的使用中比如A需要B对象传递信息，那么A就需要代理(delegate)，则在A的.h文件中暴露`RACSubject `对象，并重写A的析构(init)方法去创建`RACSubject`对象，在A的.m文件中去订阅自己的`RACSubject `对象。
那么在外部B只需要对A.subject使用`sendNext `即可传递信息并促发预先写好的`didSubscribe`闭包。