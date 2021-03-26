---
layout: post
title: UIAccessibility初见
tags:
  - 辅助功能
  - Voice Over
  - 朝花夕拾
categories:
  - iOS学习笔记
date: 2016-12-05 23:20:00
---

# UIAccessibility

`UIAccessibility`是 iOS 下`UIKit`框架中的辅助功能(如 Voice Over)的一套功能。最近在项目中曾经使用了一些这部分的功能，姑且把自己学到的一点皮毛写在这里。

## UIAccessibilityTraits

`accessibilityLabel`和`accessibilityHint`这些也就不多费口舌了。我们直接来讲`accessibilityTraits`，这个属性表示在`Voice Over`下我们访问一个元素的时候，它将被作为哪种类型的对象对待。
`UIAccessibilityTraits`是一个数值类型，它如下的所有类型都在`UIAccessibilityConstants.h`中可以找到

```swift
// Used when the element has no traits.
public var UIAccessibilityTraitNone: UIAccessibilityTraits

// Used when the element should be treated as a button.
public var UIAccessibilityTraitButton: UIAccessibilityTraits

// Used when the element should be treated as a link.
public var UIAccessibilityTraitLink: UIAccessibilityTraits

// Used when an element acts as a header for a content section (e.g. the title of a navigation bar).
@available(iOS 6.0, *)
public var UIAccessibilityTraitHeader: UIAccessibilityTraits

// Used when the text field element should also be treated as a search field.
public var UIAccessibilityTraitSearchField: UIAccessibilityTraits

// Used when the element should be treated as an image. Can be combined with button or link, for example.
public var UIAccessibilityTraitImage: UIAccessibilityTraits

/*
 Used when the element is selected.
 For example, a selected row in a table or a selected button within a segmented control.
 */
public var UIAccessibilityTraitSelected: UIAccessibilityTraits

// Used when the element plays its own sound when activated.
public var UIAccessibilityTraitPlaysSound: UIAccessibilityTraits

// Used when the element acts as a keyboard key.
public var UIAccessibilityTraitKeyboardKey: UIAccessibilityTraits

// Used when the element should be treated as static text that cannot change.
public var UIAccessibilityTraitStaticText: UIAccessibilityTraits

/*
 Used when an element can be used to provide a quick summary of current
 conditions in the app when the app first launches.  For example, when Weather
 first launches, the element with today's weather conditions is marked with
 this trait.
 */
public var UIAccessibilityTraitSummaryElement: UIAccessibilityTraits

// Used when the control is not enabled and does not respond to user input.
public var UIAccessibilityTraitNotEnabled: UIAccessibilityTraits

/*
 Used when the element frequently updates its label or value, but too often to send notifications.
 Allows an accessibility client to poll for changes. A stopwatch would be an example.
 */
public var UIAccessibilityTraitUpdatesFrequently: UIAccessibilityTraits

/*
 Used when activating an element starts a media session (e.g. playing a movie, recording audio)
 that should not be interrupted by output from an assistive technology, like VoiceOver.
 */
@available(iOS 4.0, *)
public var UIAccessibilityTraitStartsMediaSession: UIAccessibilityTraits

/*
 Used when an element can be "adjusted" (e.g. a slider). The element must also
 implement accessibilityIncrement and accessibilityDecrement.
 */
@available(iOS 4.0, *)
public var UIAccessibilityTraitAdjustable: UIAccessibilityTraits

// Used when an element allows direct touch interaction for VoiceOver users (for example, a view representing a piano keyboard).
@available(iOS 5.0, *)
public var UIAccessibilityTraitAllowsDirectInteraction: UIAccessibilityTraits

/*
 Informs VoiceOver that it should scroll to the next page when it finishes reading the contents of the
 element. VoiceOver will scroll by calling accessibilityScroll: with UIAccessibilityScrollDirectionNext and will
 stop scrolling when it detects the content has not changed.
 */
@available(iOS 5.0, *)
public var UIAccessibilityTraitCausesPageTurn: UIAccessibilityTraits
```
