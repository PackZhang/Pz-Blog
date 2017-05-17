---
title: ReactiveCocoa之核心信号---RACSignal
date: 2017-05-17 14:48:46
tags: ReactiveCocoa
---
### 创建信号

```objectivec
RACSignal *signal = [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
       
        //被订阅时触发闭包
        //发送信号给订阅者
        [subscriber sendNext:@1];
        [subscriber sendNext:@2];
        [subscriber sendNext:@3];
        
        [subscriber sendCompleted];
        
        return [RACDisposable disposableWithBlock:^{
            NSLog(@"信号销毁");
        }];
    }];
```

`createSignal `最终会调用到以下`RACDynamicSignal`类的对象方法

```objectivec
+ (RACSignal *)createSignal:(RACDisposable * (^)(id<RACSubscriber> subscriber))didSubscribe {
	RACDynamicSignal *signal = [[self alloc] init];
	signal->_didSubscribe = [didSubscribe copy];
	return [signal setNameWithFormat:@"+createSignal:"];
}
```

可以看出这个方法做的事情非常简单，创建Signal对象，并将didSubscribe代码块保存起来。

> 值得注意的是每一个信号的终点都是complete或者error，否则下一次对信号订阅操作不会生效。

### 订阅信号
```objectivec
	//订阅信号
    [signal subscribeNext:^(id  _Nullable x) {
        //接收到信号时触发闭包
        NSLog(@"%@",x);
    }];
```
`subscribeNext `方法对刚才创建的signal进行订阅操作，`subscribeNext `的实现如下

```objectivec
- (RACDisposable *)subscribeNext:(void (^)(id x))nextBlock {
	NSCParameterAssert(nextBlock != NULL);
	
	RACSubscriber *o = [RACSubscriber subscriberWithNext:nextBlock error:NULL completed:NULL];
	return [self subscribe:o];
}

- (RACDisposable *)subscribe:(id<RACSubscriber>)subscriber {
	NSCParameterAssert(subscriber != nil);

	RACCompoundDisposable *disposable = [RACCompoundDisposable compoundDisposable];
	subscriber = [[RACPassthroughSubscriber alloc] initWithSubscriber:subscriber signal:self disposable:disposable];

	if (self.didSubscribe != NULL) {
		RACDisposable *schedulingDisposable = [RACScheduler.subscriptionScheduler schedule:^{
			RACDisposable *innerDisposable = self.didSubscribe(subscriber);
			[disposable addDisposable:innerDisposable];
		}];

		[disposable addDisposable:schedulingDisposable];
	}
	
	return disposable;
}

```
内部会创建订阅者并且保存`nextBlock `代码快中的代码，并且进入到`RACDynamicSignal `的`subscribe `方法中执行在创建信号时保存的`didSubscribe `代码块中保存的代码。
在`didSubscribe `中已经写好了`[subscriber sendNext:@1];`方法，`sendNext`的实现如下

```objectivec
- (void)sendNext:(id)value {
	@synchronized (self) {
		void (^nextBlock)(id) = [self.next copy];
		if (nextBlock == nil) return;

		nextBlock(value);
	}
}
```
在执行`sendNext`方法时会执行在订阅信号时保存的`nextBlock `代码，将数字1打印出来。