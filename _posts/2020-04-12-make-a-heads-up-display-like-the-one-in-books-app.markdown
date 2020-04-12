---
layout: post
title: Make a Heads-Up Display (HUD) Like the One in Books App
---

The HUD is a common component in iOS and macOS. Using it everywhere will hinder the user from using our app because of its big size. But it's still a good way to inform the user visually if we use it judiciously.

Here is an example in Books app:

| <img src="/assets/images/books-hud.png" alt="HUD in Books app" width="200"/> |
|:-:|
| HUD in Books app |

But iOS doesn't provide an out-of-the-box feature to show a HUD. So let's try to make one by ourselves.

## UIAlertController

The background shape of the HUD is just like the shape of an alert, so let's try to implement it using the `UIAlertController`.

``` swift
// Set an empty string to create an empty UIAlertController
let alertController = UIAlertController(title: "", message: nil, preferredStyle: .alert)
let alertRootView = alertController.view!
alertRootView.addSubview(contentView)

NSLayoutConstraint.activate ([
    contentView.leadingAnchor.constraint(equalTo: alertRootView.leadingAnchor, constant: 0),
    contentView.trailingAnchor.constraint(equalTo: alertRootView.trailingAnchor, constant: 0),
    contentView.topAnchor.constraint(equalTo: alertRootView.topAnchor, constant: 0),
    contentView.bottomAnchor.constraint(equalTo: alertRootView.bottomAnchor, constant: 0),
])

present(alertController, animated: true)
```

| <img src="/assets/images/alert-hud.png" alt="HUD by UIAlertController" width="200"/> |
|:-:|
| HUD by `UIAlertControler` |

It just works, and we can dismiss it by a `Timer`:

``` swift
Timer.scheduledTimer(withTimeInterval: 1, repeats: false) { [weak self] _ in
    self?.dismiss(animated: true)
}
```

It's not bad, but here are two things we can improve to minimize the disruption to the user like Books app.

- Unnecessary dim background outside of the HUD
- User cannot interact until the `UIAlertControler` dismissed

As the name of `UIAlertController`, it's designed to interrupt the user. Let's try another approach.

## addSubview

It's more controllable to add the HUD as a subview. We can add it to the current or the root `UIViewController` or even to the current `UIWindow` as we want.

However, with this approach, we have to implement the background of the HUD.

Let's create a blurred background first.

``` swift
let blurEffect = UIBlurEffect(style: .systemMaterial)
let visualEffectView = UIVisualEffectView(effect: blurEffect)
```

Then configure the corner of it.

``` swift
visualEffectView.clipsToBounds = true
let layer = visualEffectView.layer
layer.cornerRadius = 14
// To make the corner more smoothly
layer.cornerCurve = .continuous
```

Finally, add content to `visualEffectView`.

``` swift
// UIVisualEffectView requires us to add subviews this way
blurEffectView.contentView.addSubview(contentView)
```

Let's check the result.

| <img src="/assets/images/books-hud.png" alt="HUD in Books app" width="200"/> | <img src="/assets/images/view-hud.png" alt="Our HUD" width="200"/> |
|:-:|:-:||
| HUD in Books app | Our HUD |

To make the HUD more delightful, let's add show/hide animations.

``` swift
let duration = 0.2
let initialAlpha = 0
let initialTransform = CGAffineTransform(scaleX: 0.8, y: 0.8)

hudView.alpha = initialAlpha
hudView.transform = initialTransform
UIView.animate(withDuration: duration, delay: 0, options: [.curveEaseOut], animations: {
    hudView.alpha = 1
    hudView.transform = .identity
})

Timer.scheduledTimer(withTimeInterval: 1, repeats: false) { _ in
    UIView.animate(withDuration: duration, delay: 0, options: [.curveEaseIn], animations: {
        hudView.alpha = initialAlpha
        hudView.transform = initialTransform
    }) { _ in
        hudView.removeFromSuperview()
    }
}
```

And to further reduce the disruption to the user, make touch events pass through the HUD view by setting its `isUserInteractionEnabled` to `false`.

It's a significant improvement if the user wants to make some gestures like scrolling.

That's it. We made a HUD just like the one in Books app. It's not hard but has many details to care for.
