---
layout: post
title: Touch Events Not Firing In UITableViewCell
---

Many reasons cause prevent touch events from firing. Such as the button is outside the bounds of superview or there is a gesture recognizer that fired first. But do you have the experience that a view's touch events work individually but will not fire in a `UITableViewCell`?

There is a button in `UITableViewCell`.

```swift
class TableViewCell: UITableViewCell {

    let button = UIButton()

    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        addSubview(button)
        ...
    }
    ...
}
```

Just a single `UIButton`, but it's touch events will not fire.

The reason is simple but may be hard to notice. Let's find out why.

Override `hitTest(_:with:)` and add a breakpoint.

```swift
override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
    let view = hitTest(point, with: event)
    return view // Breakpoint
}
```

We will notice that the view is a `UITableViewCellContentView`. We added the button directly to `UITableViewCell` instead of it's `contentView`.

This is indeed a fundamental mistake, but if you invest it in some other way first like me, it will take some unnecessary time. If there is something wrong with touch events, let's try overriding `hitTest(_:with:)` of the view's superview first.
