---
layout: post
title: supportedInterfaceOrientations
---

To change the supported orientations for a `UIViewController`, we can override the `supportedInterfaceOrientations`. However, it's not always working as we expected in some situations.

## shouldAutorotate

> The system only calls this method if the view controller's shouldAutorotate method returns true.

According to the documentation, override `shouldAutorotate` to false will cause `supportedInterfaceOrientations` not working. However, if we only plan to support only one orientation, it seems alright to override `shouldAutorotate` to false.

## Container View Controller

> When the user changes the device orientation, the system calls this method on the root view controller or the topmost presented view controller that fills the window.

When displaying a view controller in a container view controller, the view controller is not presented by the container view controller, but just embedded in the container view controller's view.

So it's not in accord with the quotation above from the documentation. Even when we are using `UINavigationController`, `UITabBarControllers` etc.

We have to override `supportedInterfaceOrientations` for the container view controller. Here is an example subclass of `UINavigationController`.

```swift
class NavigationController: UINavigationController {

    override var supportedInterfaceOrientations: UIInterfaceOrientationMask {
        topViewController?.supportedInterfaceOrientations ?? .all
    }
}
```

## modalPresentationStyle

> the topmost presented view controller that fills the window

The topmost presented view controller needs to fill the window to make `supportedInterfaceOrientations` to take effect.

So instead of using the default `modalPresentationStyle` from iOS 13-`pageSheet`, we have to use some styles that fill the window, such as `fullScreen`.

## Undocumented Behaviors(?)

With three points should be enough for us to configure `supportedInterfaceOrientations`. But when we are using `overFullScreen`, there will be more unexpected behaviors that I couldn't find related description from the documentation.

The first one is, when we present a `portrait` view controller from an `all` view controller by `overFullScreen`, the result will be the same as by `fullScreen`. However, when we present an `all` view controller from a `portrait` view controller by `overFullScreen`, the presented view controller can't rotate.

I tried some pattern and found out that, `supportedInterfaceOrientations` only works when `supportedInterfaceOrientations` of the presented view controller is less than the presenting view controller.

The second one is more like a bug. If we present an `all` view controller by `overFullScreen`, and then present a `portrait` view controller by `fullScreen`, the third view controller will be able to rotate.

I have no idea why this will happen. And to avoid this behavior we have to override `supportedInterfaceOrientations` for the root view controller to find the proper child and use it's `supportedInterfaceOrientations`.

## Conclusion

Override `supportedInterfaceOrientations` can change the supported orientations for a `UIViewController`, but it has some requirement to fulfill and may have some strange behaviors in some situations. Instead of relying on the default behaviors of it, we may consider overriding `supportedInterfaceOrientations` of the root view controller to define the specific behaviors we want.
