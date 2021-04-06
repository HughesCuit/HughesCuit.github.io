---
layout: post
title: ReactiveCocoa(RAC)
date: 2015-11-09 04:45:39
tags: 技术学习
categories: 开发笔记
---

## -- 基于响应式编程的第三方框架

### 函数响应式编程 (FRP)

    一种和事件流有关的编程模式，关注导致状态值改变的行为事件，一系列的事件组成了事件流。

<!--more-->

#### 思想示例: C = A + B （编程代码）

    使用FRP ：C值依赖于A和B的值
    程序运行过程中：
    1.初始化 A = 1 ， B = 2 时 C = 3
    2.程序运行过程中，若A值发生改变 A =  2 ，那C也将对应改变   C = 4
    3.运行过程中，若B值发生改变 B = 5 ，那C 对应改变  C = 7

### RAC 试图解决的问题

- 传统 iOS 开发过程中，状态以及状态之间依赖过多的问题
- 传统 MVC 架构的问题：Controller 比较复杂，可测试性差
- 提供统一的消息传递机制

### RAC 核心概念

- Signal(信号)
  - Completer(完成)
  - Error(错误)
- Subscribe(订阅)
  - Next（下一步)

### RAC 基本使用

```objc
    UITextField *textField = [[UITextField alloc] init];
    //创建文本信号并订阅，若产生信号则执行next事件(block内容)
    [textField.rac_textSignal subscribeNext:^(id x) {
        //code...
    }];
```

### RAC Next 事件的限制和内容转化

- Map(转化)
  可以将信号内容转化为任意对象
- Filter(过滤)
  设置 Next 执行的要求

### RAC 相关资源链接

- 这样好用的 ReactiveCocoa，根本停不下来
  [http://www.cocoachina.com/ios/20150817/13071.htm](http://www.cocoachina.com/ios/20150817/13071.htm)
- ReactiveCocoa2 实战
  [http://www.cocoachina.com/industry/20140609/8737.html](http://www.cocoachina.com/industry/20140609/8737.html)
- ReactiveCocoa2 源码浅析
  [http://www.cocoachina.com/ios/20150827/13252.html](http://www.cocoachina.com/ios/20150827/13252.html)
- ReactiveCocoa 底层实现解析
  [http://blog.sunnyxx.com/tags/Reactive%20Cocoa%20Tutorial/](http://blog.sunnyxx.com/tags/Reactive%20Cocoa%20Tutorial/)
- Limboy 博客，内容质量高
  [http://limboy.me](http://limboy.me)
- 细说 ReactiveCocoa 的冷信号与热信号(系列文章)
  [http://tech.meituan.com/talk-about-reactivecocoas-cold-signal-and-hot-signal-part-1.html](http://tech.meituan.com/talk-about-reactivecocoas-cold-signal-and-hot-signal-part-1.html)
