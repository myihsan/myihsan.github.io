---
layout: post
title: Is There a Common Way to Describe Shadows for Developers?
---

We can create shadows effortlessly by a design tool by setting the color, offset, and blur radius or even spread radius if we are using Sketch.

But how can we describe our shadows to developers?

Let's take a look at some ways that developers implement shadows.

## Front-end

- [`box-shadow`](https://developer.mozilla.org/en-US/docs/Web/CSS/box-shadow)
- [`drop-shadow()`](https://developer.mozilla.org/en-US/docs/Web/CSS/filter-function/drop-shadow)

The two ways above to implement shadows can set the color, offset,  blur radius, and spread radius, so implementing a shadow on front-end development is just the same as we create one by a design tool.

## iOS

- [`CALayer`](https://developer.apple.com/documentation/quartzcore/calayer)

Every `UIView`–the base UI component has a `CALayer` and we can configure the shadow by the `CALayer`. But there is no property to set the spread radius.

Luckily for us, the spread radius will only affect the size of the shadow, and we can control it by `shadowPath`.

```swift
extension CALayer {
    func configureShadow(color: CGColor, offset: CGSize, blurRadius: CGFloat, spreadRadius: CGFloat) {
        self.shadowColor = color
        self.shadowOffset = offset
        self.shadowRadius = blurRadius

        if spreadRadius == 0 {
             // Use a standard shadow shape, which matches the shape of this layer
            self.shadowPath = nil
        } else {
            let inset = -spread
            let rect = bounds.insetBy(dx: inset, dy: inset)
            shadowPath = UIBezierPath(roundedRect: rect, cornerRadius: cornerRadius).cgPath
        }
    }
}
```

With this extension, we can create the same shadow of the design on iOS.

## Android

- [`setElevation`](https://developer.android.com/reference/android/view/View#setElevation(float))

Elevation? It doesn't look like something related to the shadow. But it's the main thing we can use to affect the shadow.

Shadow in Android is adopted from Material Design, which has strict rules for the shadow[^1].

- Color: [`setOutlineAmbientShadowColor`](https://developer.android.com/reference/kotlin/android/view/View#setOutlineAmbientShadowColor(kotlin.Int)) and [`setOutlineSpotShadowColor`](https://developer.android.com/reference/kotlin/android/view/View#setOutlineSpotShadowColor(kotlin.Int)) (Only avaliable from Android 9)
- ~~Offset~~: Determined by the key(spot) light
- Blur radius: Affected by the `elevation`
- ~~Spread radius~~: Determined by key(spot) and ambient lights

As we can see the elevation is almost the only thing we can affect the shadow. But the elevation also determines the hierarchy of objects, so there is no way to set a bigger blur radius for a lower object by elevation.

## Flutter

- [`BoxShadow`](https://api.flutter.dev/flutter/painting/BoxShadow-class.html)

We can infer from the name that we can set all property for shadows as we do on the front-end. However, we cannot set those properties directly.

A common way is to wrap the object by `Container` and set `BoxShadow` to the `Container`.

```dart
Container(
    decoration: BoxDecoration( // Or a ShapeDecoration
        ...
        shadows: [BoxShadow(...)],
    )
)
```

## Conclusion

> But how can we describe our shadows to developers?

In most of the cast, describing our shadows as we described them on the design tools is enough.

As for Android developers, let's create shadows adopted from Material Design for them. Or give them a break by ignoring some detail of shadows.

## References

- [How to control shadow spread and blur?](https://stackoverflow.com/questions/34269399/how-to-control-shadow-spread-and-blur)

***

[^1]: [Shadow in Material Design]({% post_url 2020-06-07-shadow-in-material-design %})
