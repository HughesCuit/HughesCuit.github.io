---
layout: post
title: Swift进阶--结构体与类（一）
tags: []
categories: []
date: 2017-01-19 18:31:00
---

# 引用类型 VS 值类型

Swift 中提供的数据类型分为**引用类型（reference type）**和**值类型（value type）**两类，其中"类"属于引用类型，而"Int"、"struct"等类型属于值类型。

## 值类型

值类型指的是在`赋值`、`传递参数`、`初始化`过程中，会创建一个新的实例的类型，在使用值类型数据的时候，我们往往需要对某些值进行计算操作，又需要保证这些值的不变性。比如下面的例子:

```swift
struct Person {
    var age:Int
}

var person1 = Person(age:1)
var person2 = person1
person2.age += 1

person1.age //1
person2.age //2

```

上面的例子中，我们有一个`Person`结构体。person1 是 1 岁，person2 比 person1 大一岁，我们将 person1 赋值给 person2，并且对 person2 的 age 做加 1 操作,结果是 person2 与 person1 的 age 并不相同，person1 的 age 没有随着 person2.age 值的变化而变化

## 引用类型

引用类型是指在操作引用类型数据时，我们实际操作的是它的引用，通过引用来间接地访问实例，一个引用类型的实例可以通过引用被持有多次，而对它生命周期的管理则使用了自动引用计数(ARC)的方式，引用类型在`赋值`、`传递参数`等过程中不会创建新的实例，而是复制了引用，这意味着，多个持有者可以共同持有一个引用类型实例，持有者之间对它的操作也会相互影响。
还是上面的例子，我们使用`class`来实现：

```swift
class Person {
    var age:Int
    init(age:Int) {
        self.age = age
    }
}

var person1 = Person(age:1)
var person2 = person1
person2.age += 1

person1.age //2
person2.age //2

```

从上面可以看出，将 person1 赋值给 person2 之后，对 person2.age 进行的操作影响了 person1，准确的说，person1 在赋值给 person2 时，并没有创建新的实例，而是将前一步创建的实例的**引用**复制了一份给 person2。

# 结构体 VS 类

在 Swift 语言中使用了大量的`struct`和`class`。前文说了，结构体和类分别是值类型和引用类型。使用结构体时，往往是需要构建不可变的值，这意味着使用结构体的部分代码可以具有线程安全的特性。因为结构体是值类型，所以一个结构体实例只会有一个持有者，所以操作结构体的时候不需要考虑引用循环。

当我们在关注一个值类型实例时，我们关注的是它的值，并且它在赋值时只能复制，但是在某些情况下，我们不仅关注值，还关注实体的同一性。也就是说，我们需要确定两个变量确实是同一个实例，而非只是值相等。用银行账户打个比方，`a`账户存款 100 元，`b`账户存款也是 100 元，我们不能确定`a`账户就是`b`账户。所以在这个地方，使用结构体来构建实例我认为不太合适，使用类可以自然的判断两个变量持有的是否是同一个实例(Swift 中使用`===`操作符判断两个变量是否引用的同一个实例)。

```swift
//类实现
class AccountClass {
    var number:Int
    var money:Int
    init(no: Int, cash: Int) {
        number = no
        money = cash
    }
}

let accountA = AccountClass(no: 101, cash: 1000)
let accountB = accountA
let accountC = AccountClass(no: 101, cash: 1000)

accountB === accountA //true
accountC === accountA //false

//结构体实现
struct AccountStruct:Equatable {
    var number:Int
    var money:Int

    public static func ==(lhs: AccountStruct, rhs: AccountStruct) -> Bool{
        return lhs.money == rhs.money && lhs.number == rhs.number
    }
}

let accA = AccountStruct(number: 101, money: 1000)
let accB = accA
let accC = AccountStruct(number: 101, money: 1000)

accB == accA //true
accC == accA //true
```

上面的例子中分别使用了类和结构体来实现银行账号的定义，我们分别构造了一个类和结构体的`A账号`，再将`A账号`赋值给`B账号`，然后使用相同的初始化方式构造一个与`A账号`相同的`C账号`。这时我们可以看到使用类实现的`accountB`和`accountA`是完全相同(引用同一实例)的。而结构体本身不能使用`==`来比较，我们实现了`Equatable`这个 protocol 来实现了`==`这个操作符，它的实现也只能通过判断结构体属性的值来进行判断。所以当我们拿“伪造”的`accC`和原来的`accA`进行比较时，结果告诉我们：它们是相同的!
