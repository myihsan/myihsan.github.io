---
layout: post
title: "WIP: Embed Framework with a Higher Development Target in iOS" 
---

We may need to embed some frameworks with a higher development target, such as a framework using in the Widget Extension. However, [it requires conditionally load by Objective-C](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/ExtensionScenarios.html#//apple_ref/doc/uid/TP40014214-CH21-SW3). Is there a way for swift?

I am still finding the best answer, but the current one is–no, it isn't.

The framework will be load at launch and requires related swift runtime of set development target.

So we have to lower the development target to the same as the application target and add available annotations for codes using the new APIs like below.

```swift
@available(iOS 14.0, *)
struct MyProvider: TimelineProvider {
    ...
}
```
