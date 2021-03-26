---
layout: post
title: Custom Keyboard Extension 的屁事博 -- 在Container App 中植入键盘(二)
tags:
  - 键盘
  - Keyboard
categories:
  - 开发日常
date: 2017-07-05 10:14:00
---

# 转

开始着手现有输入法植入工作，亟待解决的问题有下面几个：

1. 系统调用输入法时**KeyboardViewController**如何加载
2. 能否(如何)将**KeyboardViewController**的视图加载到应用程序的视图控制器
3. **Keyboard**如何(是否需要)与输入框进行关联

下面来一一解决：

## 系统调用键盘

上一张经典图
![](https://ws1.sinaimg.cn/large/6b7d44cfgy1fh7yo4r6z6j21170s9q3z.jpg)

关于`Extension`、`Container App`、`Host App`这三者的关系这里不再赘述，网上很多资料。这里体现了一件事情就是：**系统在调用键盘时，Keyboard 是没有加载在应用程序 keyWindow 中的，**而是在一个新的名叫`UIRemoteKeyboardWindow`的窗口中

![系统日语输入法在App中加载](https://ws1.sinaimg.cn/large/6b7d44cfgy1fh7zfyf2lsj20qx0ifdi8.jpg)

## 应用程序视图中加载键盘

在以前的项目中，当我们希望系统弹出键盘时一般使用`textField.becomeFirstResponder()`这样的方法，让输入框成为第一响应者来实现。现在我们需要的是指定一个我们项目中已经实现过的`KeyboardViewController`来弹出，而不是通过系统调用键盘，然后让用户切换过去。

调查过后锁定了几个东西：`UITextField.inputView`、`UIResponder.inputViewController`、
实验发现，不论是设置`inputView`为`KeyboardViewController`的`view`或者是重载`UITextField`的`inputViewController`设置为`KeyboardViewController`都可以实现在`UIRemoteKeyboardWindow`窗口中调起该键盘。

但这样调起的键盘无法在画面中获得，也无法设置画面约束等，仅仅是一个调起指定实现的键盘的方法，局限性太大。

既然`KeyboardViewController`的父类`UIInputViewController`是基于`UIViewController`的，那么我们能不能直接将它作为**子 VC**并将视图加入应用程序视图中呢?

好像没啥问题

最简单实现`childViewController`的方法就是直接使用`storyboard`中的`Container View`。拖上容器，设置约束，将子 VC 类型设置为我们的`KeyboardViewController`，直接运行，键盘就躺在画面中等着我们了。

## 关联输入框

这时候，点击输入框，诶？

![](http://ww1.sinaimg.cn/large/6b7d44cfgy1fh88zea8k3j20r41h8jup.jpg)

怎么又弹出一个新键盘出来了，先不管，点击收起弹出来的键盘，在我们的子 VC 的键盘上操作，诶？好像可以直接输入到输入框上？！似乎：**加载在视图中的 UIInputView**，输出文字会自动输入到当前的`FirstResponder`中，而不用做其他关联

# 合

这似乎是一个比较好的实现，不过我们还得解决一下输入框获得焦点时系统弹出键盘的问题。这时，我们的`inputView`

```swift
	textField.inputView = UIView()
```

或者直接重载`inputViewController`、`inputView`

```swift

extension UITextField {
	//重载应用中所有textField的inputViewController
    open override var inputViewController: UIInputViewController?{
        get{
            return UIInputViewController()
        }
    }
}

class MyTextField: UITextField {
    lazy var emptyInputView = UIView()
    //重载inputView
    override var inputView: UIView? {
        set{

        }
        get{
            return emptyInputView
        }
    }
}

```

![](http://ww1.sinaimg.cn/large/6b7d44cfgy1fh8azll0zij20r41h840n.jpg)

完美 √
