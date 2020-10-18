---
layout: post
title: "Programmer Oriented UI Design: SwiftUI"
---

Last week we talked about how mank works we have to do with a simple cell. To let programmers understand our design, we still have to deliver images for all possible layout conditions. Why don't we just describe our design in some way? SwiftUI is here for us.

Let's go through the [design process]({% post_url 2019-11-03-viewmodel-a-superhero-belongs-to-androidx %}) of our cell using SwiftUI.

## Preparation

First, Download [Swift Playgrounds](https://apps.apple.com/jp/app/swift-playgrounds/id1496833156?l=en&mt=12) or [Xcode](https://apps.apple.com/jp/app/xcode/id497799835?l=en&mt=12) on the App Store and create a new playground.

- Swift Playgrounds: File > New Blank Playground
- Xcode: File > New > Playground... > Blank

Then copy the code below.

```swift
import SwiftUI
import PlaygroundSupport

PlaygroundPage.current.setLiveView(Cell())
```

With them, we can preview our `Cell` after we clicking the ▶︎ button at the bottom-right (bottom-left when using Xcode).

Before we describe our `Cell`, let's describe the title and the label of it.

```swift
struct CellTitle: View {
    var text: String

    var body: some View {
        Text(text)
            .font(.system(size: 17))
            .foregroundColor(Color.black)
    }
}

struct CellLabel: View {
    var text: String

    var body: some View {
        Text(text)
            .font(.system(size: 17))
            .foregroundColor(Color(.sRGB, red: 60 / 255, green: 60 / 255, blue: 67 / 255, opacity: 0.6))
    }
}
```

We describe them with a font and a foreground color, and we provide a parameter named text to let us set any text to our title and label.

## Design Process

Let's describe our `Cell`.

| ![Common cell design](/assets/images/common-cell-design.png) |
|:-:|
| Common cell design |

```swift
struct Cell: View {
    var body: some View {
        ZStack {
            HStack {
                CellTitle(text: "Title")
                Spacer()
            }
            HStack {
                Spacer()
                CellLabel(text: "Label")
            }
        }
        .padding(.horizontal, 16)
        .frame(maxWidth: .infinity, minHeight: 44)
        .background(Color.white)
    }
}
```

It's a little complicated, so let's talk about them separately.

```swift
.padding(.horizontal, 16)
.frame(maxWidth: .infinity, minHeight: 44)
.background(Color.white)
```

This part is simple, we describe the horizontal padding, the minimum height, and the background color for the `ZStack` just like we described the font and the foreground color for the `Text`.

```swift
ZStack {
    HStack {
        CellTitle(text: "Title")
        Spacer()
    }
    HStack {
        Spacer()
        CellLabel(text: "Label")
    }
}
```

We describe our title and label in an `HStack` (Horizontal Stack) with a `Spacer` which will take the rest space of the `HStack` to make the title aligns to the leading edge, and the label aligns to the trailing edge.

And finally, we put those two `HStack`s in a `ZStack`, so that they can stack on the z-axis.

You may curious why we don't put them in one `HStack`. You are right, but the description of the `Cell` is trying to simulate the behavior when we design in a design tool.

Allowing components to stack on the z-axis is the default behavior in a design tool, but when it comes to SwiftUI, it so weird that we may never try to describe our design this way. Here is what it should be in the first place.

```swift
HStack {
    CellTitle(text: "Title")
    Spacer()
    CellLabel(text: "Label")
}
```

Let's make the content area of our title and label visible.

| ![Visible content area](/assets/images/visible-content-area.png) |
|:-:|
| Visible content area |

```swift
HStack {
    CellTitle(text: "Title")
        .background(Color.green)
    Spacer()
    CellLabel(text: "Label")
        .background(Color.yellow)
}
```

We are not required to do this, but before we get familiar with SwiftUI, it helps us to understand the content area of our title and label.

Next, let's describe the maximum width of them.

| ![Explicit content area](/assets/images/explicit-content-area.png) |
|:-:|
| Explicit content area |

``` swift
HStack {
    CellTitle(text: "Title")
        .frame(maxWidth: .infinity, alignment: .leading)
        .background(Color.green)
    Spacer(minLength: 16)
    CellLabel(text: "Label")
        .frame(maxWidth: 65, alignment: .trailing)
        .background(Color.yellow)
}
```

This time we describe the maximum width for our title and label. The label has an explicit maximum width and the title has an infinity maximum width means it will take the rest space of the `HStack` as possible as it can. Last, we describe the minimum width for the `Spacer` to separate our title and label with 16 points.

When we plan to truncate the title in one line.

| ![Truncating text](/assets/images/truncating-text.png) |
|:-:|
| Truncating text |

```swift
CellTitle(text: "A Long Enough Title That Will Take Two Lines")
    .lineLimit(1)
    ...
```

This is the default behavior of SwiftUI, so all we need to do is to limit the line of the title.

When we plan to adjust the font size for the width.

| ![Autosizing text](/assets/images/autosizing-text.png) |
|:-:|
| Autosizing text |

```swift
CellTitle(text: "A Long Enough Title That Will Take Two Lines")
    .lineLimit(1)
    .minimumScaleFactor(0.5)
    ...
```

Just set a minimum scale factor for our title.

But since the font of the title is smaller than the label, it may be better to align the title and the label by their baseline.

```swift
HStack(alignment: .lastTextBaseline) {
    ...
}
```

When we plan to display the title in multiple-line.

| ![Extendable content area](/assets/images/extendable-content-area.png) |
|:-:|
| Extendable content area |

```swift
HStack {
    ...
}
.padding(.vertical, 10)
...
```

Without `lineLimit(_:)`, all we have to do is to define the vertical padding for the `HStack`.

## Conclusion

Design by the code is not that hard as we thought before, thanks to SwiftUI. All we have to do is to describe our design in some special syntax.

The complete code is [here](https://github.com/myihsan/ProgrammerOrientedUIDesign).

Except for SwiftUI, many tools have declarative syntax to help us describe our design, such as Flutter and Jetpack Compose. So we may consider using the tool that matches us or the tool that is also using by the programmers we oriented.
