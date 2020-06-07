---
layout: post
title: Shadow in Material Design
---

Shadow is an essential component in Material Design, and it's not a decoration for style but a visual cue for the UI.

This week, let's take a look at the detail of the shadow in Material Design.

## Light Sources

> Shadows in the Material environment are cast by a key light and ambient light.

As the description in [Light and shadows](https://material.io/design/environment/light-shadows.html), we have to combine shadow from key and ambient lights to create the shadow defined in Material Design.

| ![Shadow cast by key light](https://lh3.googleusercontent.com/ZTiXsXY2gZarUOUhibnCSBZhLo_CLRY0b-2cxAddw0vqFcxiEqRhchAs0DSH63Rx_0IX_DiTvinmOFhsl2fYa8F6OX3iV1en_3L98g=w1064-v0) | ![Shadow cast by ambient light](https://lh3.googleusercontent.com/X8foa5lJrWUFmWUpGOjJnrPovBEbcEnJl5l7no185n_iK75CMVmpttZYkTyfG95w_j3nM-sippJUH9GfQ1059nxqAtKkKpQBSNHj=w1064-v0) | ![Combined shadow from key and ambient lights](https://lh3.googleusercontent.com/rZkPi0lJSr5wQoyxq9qIR2AuIjTM2PBhGOMyYS-xxdds6xI8MTfN7mffWcTMP7KK5UO9iLvvDq202nZOyKDOY5YNcaH2QX8z0UqIiQ=w1064-v0) |
|:-:|:-:|:-:|
| Shadow cast by key light | Shadow cast by ambient light | Combined shadow from key and ambient lights |

We don't have to implement the shadow on Android. The shadow is created according to the elevation of the view. And we can use [Material Components](https://github.com/material-components) to set the shadow by the elevation on iOS and Flutter.

But we are not that lucky on the web and the design. On the web, we can only set the elevation from 0 to 24[^1]. On the design, if we use the design kit in [official resource](https://material.io/resources), we only get the shadows of the [default elevations](https://material.io/design/environment/elevation.html#default-elevations).

Of course, the elevations from 0 to 24 should be enough for most of the cases, and we can get the rest of the shadows from the source code of the web.

## Elevation

We have already known that we can affect the shadow by modifying the elevation. But why?

![Combined shadow from key and ambient lights](/assets/images/elevation-shadow.png)

It's just like a shadow in the real world. The closer the object is to the light, the bigger the shadow.

So the user can understand the view hierarchy by the shadow instinctively.

## Position

Like the shadow can be affected by the elevation, it also can be affected by the position. Let's take a look at another example which has a light comes in a 45° angles[^2].

![Combined shadow from key and ambient lights](/assets/images/position-shadow.png)

We can see the size of the shadow may not change a lot, but since the position of the shadow changed, the visible part of the shadow is different.

This behavior is implemented on Android and Flutter[^3]. We even can be aware of the change of the shadow when scrolling the view, which makes the shadow more real to the user.

Unfortunately, we don’t get this behavior on iOS, web, and design. Indeed we can assume that the light is big or far away enough like the sun so that the shadow is almost not affected by the position. But in that case, the shadow is also almost not affected by the elevation.

## Color

It's normal to use dark shadows to express elevation in a light surface, but sometimes we may want to use light glows in a dark surface.

However, Material Design prohibits this approach.

> In a dark theme, shadows remain dark to accurately represent a cast shadow.

This principle may be why we can't change the color of the shadow created by elevation. But we can do it by setting the [`shadowColor`](https://api.flutter.dev/flutter/material/Material/shadowColor.html) property of the `Material`.

## Implementation

If there is a simple way to calculate the shadow from elevation, we can implement the shadow by ourselves to overcome the limitation of Material Components and the official design kit.

Since the shadow is cast by a key light and ambient light, we may expect that the shadow is created by two box shadows. However, if we check the source code of the Material Components for the web or the official design kit, we will found out that it's not that simple as we thought since it's created by three box shadows.

We may indeed be able to read the source code of Android or Flutter to figure out the algorithm of creating the shadow. But even if we know how to calculate the shadow, it’s complicated or impossible to implement in other platforms and create a plugin for our design tool, especially the part changing the shadow according to the position of the view.

## Conclusion

The shadow in Material Design is designed to reflect the real world, so it has many rules to keep, but for the same reason, it's more authentic, and I even think to implement it is romantic.

However, since this kind of shadow is difficult to implement, if we are not developing a native Android app or a Flutter app, we should think carefully if we actually need this kind of shadow.

## References

- [Design - Material Design](https://material.io/design)

***

[^1]: [Elevation - Material Components for the Web](https://material.io/develop/web/components/elevation/)

[^2]: [Making Material Design](https://www.youtube.com/watch?v=rrT6v5sOwJg&feature=youtu.be&t=3m27s)

[^3]: In Flutter 1.17.2, there may be a bug that caused a square gap in the shadow if the view is height enough.
