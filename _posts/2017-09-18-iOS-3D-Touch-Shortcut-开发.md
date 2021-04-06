---
layout: post
title: iOS 3D Touch Shortcut 开发
date: 2017-09-18 17:12:42
tags:
  - 3D Touch
categories:
  - 开发笔记
---

# ShortcutItem

`ShortcutItem`是苹果在`iOS9`/`iPhone6s`发布时添加的新功能，主要利用了新 iPhone 的 3D Touch 特性，用户在重压 APP 图标时，可以弹出设定好的应用程序快捷方式

# 配置 ShortcutItem

配置`ShortcutItem`有两种方式：通过工程`info.plist`配置文件进行添加和在 APP 的`UIApplication`对象中`shortcutItems`属性中修改。笔者下面将对其逐一介绍。

## info.plist 配置

在`info.plist`中添加如下的配置:

```xml
<key>UIApplicationShortcutItems</key>
    <array>
        <dict>
            <!--  项目Bundle中的Image Name名称，对应Shortcut右侧的图标，注意，建议使用单色、透明的png -->
            <key>UIApplicationShortcutItemIconFile</key>
            <string>open-favorites</string>
            <!-- 必须指定，Shortcut的标题，支持i18n -->
            <key>UIApplicationShortcutItemTitle</key>
            <string>Favorites</string>
            <!-- 必须指定，用于区别ShortcutItem -->
            <key>UIApplicationShortcutItemType</key>
            <string>com.mycompany.myapp.openfavorites</string>
            <!-- userinfo 可以带相应的自定义信息，以字典形式传入 -->
            <key>UIApplicationShortcutItemUserInfo</key>
            <dict>
                <key>key1</key>
                <string>value1</string>
            </dict>
        </dict>
        <dict>
            <!-- Shortcut的图标，如果没有设置IconFile可以选择系统预设的图标，种类还算比较多
            具体参见 https://developer.apple.com/documentation/uikit/uiapplicationshortcuticontype -->
            <key>UIApplicationShortcutItemIconType</key>
            <string>UIApplicationShortcutIconTypeCompose</string>
            <key>UIApplicationShortcutItemTitle</key>
            <string>New Message</string>
            <!-- 可以指定Shortcut的子标题 -->
            <key>UIApplicationShortcutItemSubtitle</key>
            <string>You Can Insert New Message</string>
            <key>UIApplicationShortcutItemType</key>
            <string>com.mycompany.myapp.newmessage</string>
            <key>UIApplicationShortcutItemUserInfo</key>
            <dict>
                <key>key2</key>
                <string>value2</string>
            </dict>
        </dict>
    </array>
```

通过上面的配置，一个程序中固定的`Shortcut`就可以配置完成了。

## 通过 UIApplication 配置

一般情况下，我们会在`AppDelegate`下，`application(_ application: didFinishLaunchingWithOptions:)`方法中添加

```swift
application.shortcutItems = []

application.shortcutItems = [UIApplicationShortcutItem(type: <#T##String#>, localizedTitle: <#T##String#>, localizedSubtitle: <#T##String?#>, icon: <#T##UIApplicationShortcutIcon?#>, userInfo: <#T##[AnyHashable : Any]?#>)]

application.shortcutItems?.append(UIApplicationShortcutItem(type: <#T##String#>, localizedTitle: <#T##String#>, localizedSubtitle: <#T##String?#>, icon: <#T##UIApplicationShortcutIcon?#>, userInfo: <#T##[AnyHashable : Any]?#>))
```

如上这样的代码即可添加。

这样做看起来比在配置文件中进行添加要快和简单许多，而且使用代码管理也显得清晰明了，但这样的坏处就是应用程序在安装或更新后，新的 ShortcutItem 需要程序先启动一次，并且执行了配置的代码后才能被添加到程序中，而使用配置文件则不需要，它在安装时就已经写入了配置。

而是用代码添加的好处是可以根据应用程序的不同状态来更改不同的`Shortcut`设置，所以实际使用中，这两种方法可能会搭配使用

# 响应 ShortcutItem 配置

上面我们为 App 添加了 ShortcutItem 配置，剩下的我们需要做的就是，根据不同的 ShortcutItem 执行不同的逻辑，我们只需要在`AppDelegate`中实现`application(_ application:performActionFor:completionHandler:`方法，在这个方法中去获取`shortcutItem`对象并执行相应逻辑即可。
