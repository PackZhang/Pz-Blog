---
title: 为应用添加Touch ID
date: 2017-05-11 15:42:41
tags: Touch ID
---

## 简介

Touch ID是苹果公司的一种指纹识别技术。苹果把用户的指纹数据存放在处理器的安全区域中，充分保护用户的数据安全。除此之外，苹果还有另外一道指纹数据安全防线，以一种前所未有的硬件技术实现了对用户数据的保护。

## 背景

最初在2013年苹果推出的iPhone 5s上出现，解锁速度稍慢。
在2015年苹果发布会上声明iPhone 6s采用了第二代Touch ID，识别速度更快。
<center>![](http://i1.piimg.com/1949/1a73de8ac30b372e.jpg)</center>
<center>相比2013年第一代Touch ID，2015年第二代Touch ID的识别面积提升了40%</center>

## 使用

    LAContext *context = [[LAContext alloc] init];
    ///从解锁屏幕成功开始的时间间隔内，去请求本地认证，将不会请求指纹验证，会直接通过验证。（最长时间5min）
    context.touchIDAuthenticationAllowableReuseDuration = _reuserTextfield.text ? [_reuserTextfield.text integerValue] : 0;

    ///失败以后的选项文字
    context.localizedFallbackTitle = @"余额指纹支付失败,去输密码";
    ///取消选项文字
    context.localizedCancelTitle = @"取消";

    ///检测是否支持指纹验证
    BOOL isTouchIDEnable = [context canEvaluatePolicy:LAPolicyDeviceOwnerAuthentication error:nil];

    if(isTouchIDEnable) {

        ///发起指纹支付
        [context evaluatePolicy:LAPolicyDeviceOwnerAuthentication localizedReason:@"余额支付" reply:^(BOOL success, NSError * _Nullable error) {

            //子线程
            NSLog(@"%d",[NSThread currentThread].isMainThread);

            dispatch_sync(dispatch_get_main_queue(), ^{

                if(success) {
                    NSLog(@"支付成功");
                    self.tipLabel.text = @"支付成功";
                    return;
                }

                switch (error.code) {
                    case LAErrorUserFallback: ///点击输入密码按钮
                    {
                        self.tipLabel.text = @"输入密码";
                        NSLog(@"输入密码");
                    }
                        break;
            });
        }];
    } else {
        NSLog(@"不能使用Touch ID");
    }

值得注意的是在指纹支付的reply代码块中的内容，默认是在子线程中执行，若有刷新UI的操作，需回到主线程执行。

## 策略

在验证与发起指纹支付的时候需要`Policy`参数，Apple提供了以下两种策略

    LAPolicyDeviceOwnerAuthentication
    LAPolicyDeviceOwnerAuthenticationWithBiometrics

两种策略的区别我以以下的流程图说明
![](http://i4.buimg.com/1949/b69eb46f844e5a54.png)
注：LAPolicyDeviceOwnerAuthentication策略在Touch ID验证失败三次以后，会要求用户输入PassCode(iPhone的开屏密码)，PassCode输错6次，iPhone将被锁住1min。三次以后，会要求用户输入PassCode(iPhone的开屏密码)，PassCode输错6次，iPhone将被锁住1min。
