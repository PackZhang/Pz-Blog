---
title: ReactiveCocoa之核心信号---RACCommand
date: 2017-05-31 10:46:52
tags: ReactiveCocoa
---

`Command`的意思为指令的意思，在使用RAC时是经常使用到类。它主要可以处理响应事件，耗时操作等。

## 给按钮添加操作
RAC使用分类给UIBarButtonItem，UIButton等控件做了支持，可以方便我们使用RAC给它们添加操作。用到的类是RACCommand。

```objectivec
@weakify(self);
    self.samplePicButton.rac_command = [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {
        @strongify(self);
        [self showSamplePic];
        
        return [RACSignal empty];
    }];
```

使用原来给按钮添加操作的方法`- (void)addTarget:(nullable id)target action:(SEL)action forControlEvents:(UIControlEvents)controlEvents;`，就会发现两个问题

1.需要额外定义方法，过程繁琐。  
2.不利于阅读，需要跳转到action的具体实现。

使用RAC提供的方法，完美解决了以上问题并且符合编码高内聚的思想。

## 执行网络操作
#### 创建命令
`RACCommand`还支持手动创建，然后在需要的时候执行。

```
marketingDataCommand = [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {
    return [self MarketingDataSignal];
}];
```

在SignalBlock中需要返回一个`RACSignal`对象，RACSignal对象封装了网络请求。

```
- (RACSignal *)MarketingDataSignal {
    return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        [MarketingRequestModel marketingDataSuccess:^(id object) {
            
            MarketingDataViewModel *vm = [[MarketingDataViewModel alloc] init];
            vm.model = object;
            self.marketingVM = vm;
            
            [subscriber sendNext:nil];
            [subscriber sendCompleted];
        } Failure:^{
            [subscriber sendCompleted];
        }];
        return nil;
    }];
}
```

以上代码可以的操作就是创建信号，信号内部调用`marketingDataSuccess `网络请求，无论成功还是失败都给订阅者`send`信息。

在外部如何去调用`RACCommand`呢？

在外部需要对`RACCommand `做两件事情，那就是`触发`与`订阅`。

#### 触发
对`RACCommand `的触发非常简单。

```
[marketingDataCommand execute:nil];
```
执行`execute `方法的时候，就会去执行`RACCommand`创建时的`signalBlock`闭包中的内容。

#### 订阅
触发`RACCommand `以后通常需要视图层有感知，例如舒刷新表格，停止动画等。订阅`RACCommand`中的信号即可。

```
[self.marketingVM.marketingDataCommand.executionSignals subscribeNext:^(RACSignal *signal) {
    [signal subscribeNext:^(id x) {
        //do something
     }];
}];
```
`executionSignals`会获取到信号的信号，在对信号进行订阅，也可以写成以下简化的形式。

```
[marketingDataCommand.executionSignals.switchToLatest subscribeNext:^(id x) {
    //do something
}];
```


#### 传递参数
在触发命令的时候(执行`- (RACSignal *)execute:(id)input`方法)，可以传入`input`参数。会作为`RACCommand`创建时的`signalBlock`闭包的参数传入。
***
在MVVM+RAC的架构中，`RACCommand `常暴露在`ViewModel`的.h文件中，以供给外部View层调用。而在`RACCommand`中的操作通常则是网络请求或者数据存储。
