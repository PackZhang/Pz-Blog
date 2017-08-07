---
title: Xcode---忽略警告
date: 2017-06-12 15:45:51
tags: Xcode
---
## 忽略警告
在iOS开发编码完成后，常常有一堆警告，可以使用系统提供的宏定义将警告忽略。

```
#pragma clang diagnostic push
#pragma clang diagnostic ignored"-Wundeclared-selector"
//希望忽略警告的代码区间
#pragma clang diagnostic pop
```

## 警告名称
警告名称可以通过警告消息在[这里](http://fuckingclangwarnings.com/)进行查找。

***
参考文章

[Xcode忽略编译警告](http://blog.csdn.net/u010462316/article/details/54618178)