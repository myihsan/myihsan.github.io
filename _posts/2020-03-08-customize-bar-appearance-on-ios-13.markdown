---
layout: post
title: Customize Bar Appearance on iOS 13
---

Since iOS 13, there are some new classes to let us customize bar appearance. We have `UINavigationBarAppearance`, `UIToolbarAppearance` and `UITabBarAppearance` which inherited from `UIBarAppearance`.

The properties we used before iOS 13 have been moved to pages named **Legacy Customizations**. However, they still work on iOS 13.

So a requestion comes to us–what should we use?

### Legacy Customizations

The first answer may be using **Legacy Customizations** since they still work on iOS 13.

If we don't need to change the appearance of `UINavigationBar` or `UITabBar` according to the `UIViewController` is showing, this solution is just fine.

But if we need, we can use `navigationItem` and `tabBarItem` in `UIViewController` to change bar appearance, only when this `UIViewController` is showing. We don't have to change bar appearance in `viewWillAppear(_:)` and recover it in `viewDidDisappear(_:)` anymore.

### UIBarAppearance

So using the `UIBarAppearance` only on iOS 13 should be the answer?

It should be the answer, but there are still some issues here that we have to fall back to **Legacy Customizations**. 

For example, `UITabBarAppearance.selectionIndicatorImage` doesn't work at this time, we have to use to old one in `UITabBar` even they have the same name.

## How to Use

The usage is similar for all kinds of bars, so let's talk about `UINavigationBarAppearance`, for example.

Let's check the [official document](https://developer.apple.com/documentation/uikit/uinavigationcontroller/customizing_your_app_s_navigation_bar) first.

- Use **Legacy Customizations** for `UINavigationBar` 🧐
- Use `UIBarAppearance` for `UINavigationItem`

I can't tell the reason why, but let's check if it works.

Assume that we want to change the background color of the navigation bar to white and only remove the shadow when the root view controller is showing.

``` swift
navigationController?.navigationBar.barTintColor = .white

let appearance = UINavigationBarAppearance()
appearance.configureWithDefaultBackground() // Have no affect to the result
appearance.shadowColor = .clear
navigationItem.standardAppearance = appearance
```

But this code will not work as expected since we set a brand new `appearance` to the `navigationItem`.

How about changing the `shadowColor` directly? It wouldn't work since `standardAppearance` is an optional property, and the default value is nil.

We can tell the official document only works for the situations that you need totally different navigation bar appearances for each view controller. Setting appearance for each view controller is not smart when we have a base appearance and want to modify it in some situations.

Let's try to fix the code above.

``` swift
navigationController?.navigationBar.barTintColor = .white

let appearance = navigationController?.navigationBar.standardAppearance.copy()
appearance.shadowColor = .clear
navigationItem.standardAppearance = appearance
```

Still not working, it seems like **Legacy Customizations** will not affect the `UIBarAppearance`. So let's implement the first line using `UINavigationBarAppearance`.

``` swift
navigationController?.navigationBar.standardAppearance.backgroundColor = .white
```

It's working! So it's possible to achieve our goal by using the `UIBarAppearance` only.

But what if we have more than one navigation controller? Do we have to write the code above for each root view controller or subclass a navigation controller?

You may already know the answer—`appearance()`. If you have tried it, you may also notice there is no `standardAppearance` or any other appearance in code completion. But it seems like an issue of code completion; we can use them normally.

``` swift
let appearance = UINavigationBarAppearance()
appearance.backgroundColor = .white
UINavigationBar.appearance().standardAppearance = appearance
```

We can't achieve it like below since this is just a proxy; the default value of `standardAppearance` is something like `nil`.

``` swift
UINavigationBar.appearance().standardAppearance.backgroundColor = .white
```

### Tips

The object of appearance will be copied when you set it to `strandardAppearance`, so the code below works as the same as above.

``` swift
navigationItem.standardAppearance = navigationController?.navigationBar.standardAppearance
navigationItem.standardAppearance?.shadowColor = .clear
```

If you find you can't modify from `appearance()` by change some properties from the bar like below.

``` swift
navigationController?.navigationBar.standardAppearance.shadowColor = .clear
```

You may have to modify it from `appearance()` directly. I have encountered this issue once in my work.

``` swift
let appearance = UINavigationBar.appearance().standardAppearance.copy()
appearance.shadowColor = .clear
navigationController?.navigationBar.standardAppearance = appearance
```

The reason seems like we modify it too early, so we actually modify the default appearance and then be overridden by the proxy.

## Summary

- Use `UIBarAppearance` only to customize bar appearance on iOS 13
- Set the appearance to the level we want and modify base on it
  - Application Level: `appearance().standardAppearance`
  - View Level (a bar shared with view controlelrs): `bar.standardAppearance`
  - View Controller Level: `item.standardAppearance`
