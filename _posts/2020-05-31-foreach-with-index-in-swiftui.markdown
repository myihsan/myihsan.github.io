---
layout: post
title: ForEach With Index in SwiftUI
---

Sometimes we may want to change the appearance of some views in `ForEach` according to its index. However, we may do it in the wrong way and even didn't notice it in the first place.

Let's going through some wrong ways and find out why it's wrong.

## Wrong Initializer

Since all we want is the index, why not we just iterate the indices?

``` swift
ForEach(array.indices) { index in
    ...
}
// or
ForEach(0..<array.count) { index in
    ...
}
```

It works[^1]. But if we need to modify the array, this way will not work.

When we try to delete some elements in the array, the app will crash with a fatal error–Index out of range. And when we try to add some elements, it will not work, and the warning will tell us the reason.

> count (1) != its initial count (0). `ForEach(_:content:)` should only be used for *constant* data. Instead conform data to `Identifiable` or use `ForEach(_:id:content:)` and provide an explicit `id`!

And the document of `ForEach(_:content:)` is telling us the same thing.

> The instance only reads the initial value of the provided data and doesn’t need to identify views across updates. To compute views on demand over a dynamic range, use init(_:id:content:).

## Wrong ID

Then let's try to use `ForEach(_:id:content:)`.

``` swift
ForEach(array.indices, id: \.self) { index in
    ...
}
```

It works when we only add or delete elements at the end of the array. But if we try to modify the array in the middle of it, we will notice the animation is weird.

The reason is we are using indices as the identifier. When we modify the array in the middle of it, we update the index of some items, but the index itself will not change, which makes the animation weird.

## Proper Way

So let's just set the proper identifier.

``` swift
ForEach(array.enumerated(), id: \.1) { index, element in
    ...
}
```

This wouldn't build since `array.enumerated()` is not a `RandomAccessCollection` which is required by `ForEach(_:id:content:)`.

We can fix this error by wrapping it with `Array` since `Array` adopted `RandomAccessCollection`.

``` swift
ForEach(Array(array.enumerated()), id: \.1) { index, element in
    ...
}
```

That's it. The animation will keep the same as we are using `ForEach(array, id: \.self)`.

## Conclusion

To use `ForEach` with index:

- Use `ForEach(_:id:content:)`
- Choice the proper identifier by something like `ForEach(Array(array.enumerated()), id: \.1)`

***

[^1]: In my experience, even we don't modify the array but only do some conditional operations in `ForEach`, it may break in some cases like after rotation.
