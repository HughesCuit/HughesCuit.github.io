---
layout: post
title: 函数式Swift——Map、Filter、Reduce
tags:
  - 函数式
  - Swift
categories:
  - iOS学习笔记
date: 2016-08-01 11:23:00
---

# 概述

本篇主要解释以及模拟实现 Map、Filter 以及 Reduce 三个方法在数组操作中的作用

> “接受其它函数作为参数的函数有时被称为高阶函数。本章中，我们将在一些来自 Swift 标准库中作用于数组的高阶函数中漫游。伴随这个过程，我们将介绍 Swift 的泛型，以及展示如何将复杂计算运用于数组。”

> 摘录来自: Chris Eidhof. “函数式 Swift”。

# 泛型介绍

Swift 中引入了泛型的概念，这个概念在其他许多高级语言中也有。使用泛型，我们可以将原本代码中很多重复的，大量相似的代码进行抽象化。
具体来说，就是可以让我们设计适用于不同类型的通用函数(函数族)，例如下图：
![image](http://ww4.sinaimg.cn/large/6b7d44cfgw1f6e1eypvlwj20vq0b73zj.jpg)
以及让我们设计泛型类，例如:`Array<Element>`、`Dictionary<Key : Hashable, Value>`、`Range<Element : ForwardIndexType>`等等。

# 模拟实现 Map

在数组中使用 map 函数的主要作用是对目标数组中的每个元素都进行某一个相同的操作，而这个进行的操作由 map 函数所接收的参数“transform: Element -> T”来规定。

```
extension Array {
    func map<T>(transform: Element -> T) -> [T] {
        var result: [T] = []
        for x in self {
            result.append(transform(x))
        }
        return result
    }
}
```

其中 Element 这个类型参数来自 Array<Element>的定义

例：当我们自己实现对某一个`Int`数组中的元素进行`+1`操作。很简单，只需要一个循环就可以完成了。

```
func increment(xs:[Int]) -> [Int]{
    var result:[Int] = []
    for x in xs {
        result.append(x+1)
    }
    return result
}
```

如果把每个元素乘 2 呢？很简单，稍微修改一下就行了。

```
func double(xs:[Int]) -> [Int]{
    var result:[Int] = []
    for x in xs {
        result.append(x*2)
    }
    return result
}
```

设想：如果我们需要对数组中每个元素进行若干种操作，是否需要将每一种操作都以这样类似的方式来实现一遍呢？

这里我们可以引入函数式编程的思想，将要进行的操作(函数)作为参数传入函数中，做出以下实现:

```
func compute(xs:[Int],transform:Int -> Int) -> [Int]{
    var result:[Int] = []
    for x in xs {
        result.append(transform(x))
    }
    return result
}
```

上面的实现，似乎还不完善，因为它只支持某一个整型数组中的元素进行整型到整型的操作，如果是其它类型的数组，或者需要将数组转换为其它类型的数组，则这个函数就不适用了。此处再引入泛型对这个函数进行改造，实现如下:

```
func compute<Element,T>(xs:[Element],transform:Element -> T) -> [T]{
    var result:[T] = []
    for x in xs {
        result.append(transform(x))
    }
    return result
}
```

至此，我们对这个函数的通用性改造就算完成了，而它与 Swift 标准库中 map 的区别在于：map 函数是协议扩展的函数，而它是顶层函数.

而它们的调用方式也不同: `array.map(transform)`、`compute(array, transform: transform`)

# 模拟实现 Filter

Filter，顾名思义，是用来过滤数组中的元素，返回的数组中包含原数组所有符合筛选条件的元素。它的原理与 Map 类似，而不同之处在于：我们传入的函数，不是用作对数组本身进行操作，而是一个筛选条件，所以传入函数的返回值为一个布尔值，用于表示该元素是否符合筛选条件。它的模拟实现为如下所示：

```
extension Array {
    func filter(includeElement: Element -> Bool) -> [Element] {
        var result: [Element] = []
        for x in self where includeElement(x) {
            result.append(x)
        }
        return result
    }
}
```

# 模拟实现 Reduce

相比于 Map、Filter，Reduce 在我看来显得更为强大，因为 Reduce 不仅可以遍历数组进行运算，而且还可以记录上一步的运算结果，对于运算结果，我们更能自由地操作它。
这里使用一个累加数组的例子来模拟 Reduce 的实现，首先是 for 循环的版本：

```
func sum(xs:[Int]) -> Int{
    var result:Int = 0
    for x in xs {
        result = result + x
    }
    return result
}
```

同理，我们引入函数式思想，将每步操作作为参数传入：

```
func sum1(xs:[Int],transform:(Int,Int) -> Int) -> Int {
    var result:Int = 0
    for x in xs {
        result = transform(result, x)
    }
    return result
}
```

接下来，我们希望操作的数组是任意类型的，操作结果也可以返回任意类型的值，所以加入泛型,由于结果是任意类型，所以初始值需要外部传入，实现如下：

```
func myReduce<Element,T>(initial:T,xs:[Element],transform:(T,Element)->T) -> T {
    var result:T = initial
    for x in xs {
        result = transform(result, x)
    }
    return result
}
```

以及 Array 扩展版本：

```
extension Array {
    func reduce<T>(initial: T, combine: (T, Element) -> T) -> T {
        var result = initial
        for x in self {
            result = combine(result, x)
        }
        return result
    }
}
```

甚至我们可以使用 Reduce 来实现 Map 和 Filter：

```
extension Array {
    func mapUsingReduce<T>(transform: Element -> T) -> [T] {
        return reduce([]) { result, x in
            return result + [transform(x)]
        }
    }
    func filterUsingReduce(includeElement: Element -> Bool) -> [Element] {
        return reduce([]) { result, x in
            return includeElement(x) ? result + [x] : result
        }
    }
}
```

[点我下载这部分的 Playground](https://pan.baidu.com/s/1c2r4Cj2)
