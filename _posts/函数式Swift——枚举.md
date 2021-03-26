---
layout: post
title: 函数式Swift——枚举
tags:
  - 函数式
  - Swift
categories:
  - iOS学习笔记
date: 2016-08-18 11:48:00
---

## 关于枚举

### OC 中的枚举与 Swift 中的枚举

`Objective-C`中的枚举类型类似于`NSASCIIStringEncoding`、`NSNEXTSTEPStringEncoding`、`NSJapaneseEUCStringEncoding`、`NSUTF8StringEncoding`酱婶儿的，用起来也是比较麻烦。

在`Swift`中，枚举出现的方式已经没有那么粗暴了，而是像`UIViewContentMode.Bottom`、`UIViewAnimationOptions.Autoreverse`这样比较温柔的方式，实际使用的时候还支持像`.Bottom`、`.Autoreverse`这样的文艺使用方法。

而且在我们实际使用枚举值的时候，枚举的定义还可以加入`关联值`、`泛型`等概念，这使得枚举的实际使用价值更加高了

## 关联值

枚举中加入关联值的作用在于：我们在传递一个枚举值的时候，还可以同时传递一个(或多个)规定的关联值，这样我们可以通过媒体传递更多的信息。
比如我们要实现一个网络请求，请求的结果有`成功`和`失败`。

我们定义一个网络请求的结果枚举：

```swift
enum Result {
    case Success
    case Failure
}
```

我们拿到结果之后，可以知道这个结果是否成功了。但是成功之后，我的网络请求返回的数据呢？失败之后，我我是为什么失败了呢？这样只能从其他方式再去获取了，能不能值传递一个结果就能拿到我想要的这些东西呢？

似乎关联值可以：

```swift
enum Result {
    case Success(String)
    case Failure(NSError)
}
```

这样我们就可以这样来使用这个结果:

```swift
func printResult(result:Result) {
    switch result {
    case .Success(let data):
        print(data)
    case .Failure(let error):
        print(error)
    }
}

let a = Result.Success("Hello World")
let b = Result.Failure(NSError(domain: "test Error", code: 404, userInfo: ["1234567":"abcdefg"]))
printResult(a) // Hello World
printResult(b) // Error Domain=test Error Code=404 "(null)" UserInfo={1234567=abcdefg}
```

看起来还不错，但是每次想要取值都去`switch-case`一下还是太麻烦了，我们将它改成`.`一下就能取值的:

```swift
enum Result {
    case Success(String)
    case Failure(NSError)

    var data: String? {
        switch self {
        case .Success(let data):
            return data
        case .Failure:
            return nil
        }
    }

    var error: NSError? {
        switch self {
        case .Failure(let error):
            return error
        case .Success:
            return nil
        }
    }
}
print("\(a.data)"+" "+"\(a.error)") //Optional("Hello World") nil
print("\(b.data)"+" "+"\(b.error)") //nil Optional(Error Domain=test Error Code=404 "(null)" UserInfo={1234567=abcdefg})
```

## 添加泛型

现在，一个网络请求的结果枚举就完成了，我现在还需要一个数据库请求的结果枚举，功能都差不多，只是请求成功后数据类型有点不一样，像下面这样在这个枚举中加入泛型就可以拿来用了：

```swift
enum Result<Data,Error:ErrorType> { //声明枚举时添加类型签名
//enum Result {
    case Success(Data)//使用泛型作为关联值
    case Failure(Error)
//    case Success(String)
//    case Failure(NSError)

    var data: Data? {//在成员和方法中也使用泛型
//    var data: String? {
        switch self {
        case .Success(let data):
            return data
        case .Failure:
            return nil
        }
    }

    var error: Error? {
//    var error: NSError? {
        switch self {
        case .Failure(let error):
            return error
        case .Success:
            return nil
        }
    }
}
```

## 数据类型中的代数学

### 同构

> “比较直观的解释是，如果两个类型 A 和 B 在相互转换时不会丢失任何信息，那么它们就是同构的。”

> 摘录来自: Chris Eidhof. “函数式 Swift”。 iBooks.

通过枚举和泛型可以定义许多代数数据类型，具有某些数学运算的特性，比如`加法`、`乘法`、`微分`、`积分`
