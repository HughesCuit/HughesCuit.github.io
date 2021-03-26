---
layout: post
title: ReactiveCocoa4基本操作
tags:
  - RAC
  - FRP
categories:
  - iOS学习笔记
date: 2016-04-06 12:06:00
---

本文主要提一些在 ReactiveCocoa4 中主要的操作，主要包括

- observe
- on
- lift
- map
- filter
- reduce
<!--more-->

## 对事件流产生副作用(Performing side effects with event streams)

### observe(观察)

**`Observer(观察者)`**可以对**`Signals(信号)`**进行**`Observe(观察)`**操作，观察这个信号将来每次发送的事件。

```swift
signal.observe(Signal.Observer { event in
  switch event {
  case let .Next(next):
   print("Next: (next)")
  case let .Failed(error):
   print("Failed: (error)")
  case .Completed:
   print("Completed")
  case .Interrupted:
   print("Interrupted")
}})
```

从上面的代码中我们可以看到，事件一共有**`Next`**、**`Failed`**、**`Completed`**、**`Interrupted`**四种。当信号发送事件时，就会触发相应的回调。设置回调的方法如下：

```swift
signal.observeNext { next in
 print("Next: (next)") }
signal.observeFailed { error in
 print("Failed: (error)")}
signal.observeCompleted {
print("Completed") }
signal.observeInterrupted {
print("Interrupted")}
```

最后，**你可以只关注你关心的事件而不必观察所有类型。**

## 注入影响（Injecting effects）

### on

通过`on`操作可以用来观察`SignalProducer`,即使没有**`订阅者`**也可以触发回调，像下面这样：

```swift
let producer = signalProducer
.on(started: {
  print("Started")
}, event: { event in
  print("Event: (event)")
}, failed: { error in
  print("Failed: (error)")
 }, completed: {
  print("Completed")
 }, interrupted: {
  print("Interrupted")
 }, terminated: {
  print("Terminated")
}, disposed: {
  print("Disposed")
 }, next: { value in
  print("Next: (value)")
 })
```

而与`Observer`观察`Signal`不同的是，`producer`需要在**started**之后才能触发。

## 操作符合成(Operator composition)

## lift

通过**`lift`**操作可以将**`Signal`**的操作符向上迁移至**`SignalProducer`**，产生一个新的`SignalProducer`，在其产生的每个`Signal`中应用该操作符。

## Map(映射)、Filter(过滤)、Reduce(聚集) --转换事件流(Transforming event streams)

### Map 映射

使用**`map`**操作可以将事件流进行转换，将原来的值进行改变等操作后产生一个新的事件流、例如：

```swift
let (signal, observer) = Signal<String, NoError>.pipe()
signal
 .map { string in string.uppercaseString }
 .observeNext { next in print(next) }
observer.sendNext("a") // Prints A
observer.sendNext("b") // Prints B
observer.sendNext("c") // Prints C
```

### Filter 过滤

**`filter`**顾名思义，可以对信号进行过滤，只传递满足条件的值。比如：

```
let (signal, observer) = Signal<Int, NoError>.pipe()
signal
.filter { number in number % 2 == 0 }
 .observeNext { next in print(next) }
observer.sendNext(1) // Not printed
observer.sendNext(2) // Prints 2
observer.sendNext(3) // Not printed
observer.sendNext(4) // prints 4
```

在上面的代码中，条件为`filter { number in number % 2 == 0 }`,表示符合条件的值为能被 2 整除的数，则`1`和`3`被过滤掉了，观察者不会观察到这两个`Next`事件。

### Aggregate 聚集

**`reduce`**可以将某一`事件`的值聚集后合成为一个新值，在输入的流结束后发送。

```
let (signal, observer) = Signal<Int, NoError>.pipe()
signal .reduce(1) { $0 * $1 }
.observeNext { next in print(next) }
observer.sendNext(1) // nothing printed
observer.sendNext(2) // nothing printed
observer.sendNext(3) // nothing printed
observer.sendCompleted() // prints 6
```

**`collect`**操作可以将一个事件流的值聚合为一个单个的数组值，在输入流结束后发送。

```
let (signal, observer) = Signal<Int, NoError>.pipe()

signal
    .collect()
    .observeNext { next in print(next) }

observer.sendNext(1)     // nothing printed
observer.sendNext(2)     // nothing printed
observer.sendNext(3)     // nothing printed
observer.sendCompleted()   // prints [1, 2, 3]
```

附上一个网址，以图案的方式展示了很多 RAC 信号的操作http://neilpa.me/rac-marbles/
更多资料：
[ReactiveCocoa 4 官方文档](https://github.com/ReactiveCocoa/ReactiveCocoa/tree/master/Documentation)
