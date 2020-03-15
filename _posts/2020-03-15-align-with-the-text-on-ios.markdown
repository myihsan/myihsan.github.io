---
layout: post
title: Align With the Text on iOS 
---

On iOS, text-to-text alignment is simple when you are using Auto Layout.

- `UIView`
  - `firstBaselineAnchor`
  - `lastBaselineAnchor`
- `UIStackView.Alignment`
  - `firstBaseline`
  - `lastBaseline`

With the properties and arguments, we can cover most of the text-to-text alignment requirements.

But how about view-to-text alignment. It's not that simple since the baseline of a non-text view is just the bottom of the view.

We need to get familiar with the font metrics.

## Font Metrics

| ![Font metrics](https://developer.apple.com/library/archive/documentation/TextFonts/Conceptual/CocoaTextArchitecture/Art/glyph_metrics_2x.png)|
|:-:|
| Font metrics[^1]|

We can get all the information above from `UIFont`. The origin `(0, 0)` is on the baseline, and the value of `descent` is negative.

## Calculation

With the information, we can align with the text as we want. Let's see an example to align the center of a text (of course, we can align to the center of the text's view if the text is a single-line text).

``` swift
extension UIFont {
    func bottomOffsetFromBaselineForVerticalCentering(targetHeight height: CGFloat) -> CGFloat {
        let textHeight = ascender - descender
        let offset = (textHeight - height) / 2 + descender
        return offset
    }
}
```

The calculation is easy, and we can adjust it as we need. Here is another example that may get a better (not the perfect) result for Chinese/Japanese characters.

``` swift
let offset = (capHeight - height) / 2
```

|![Compare Latin letters with Kanji and Kana](https://theplant.jp/system/media_libraries/50/file.20190510023746.png?time=1557455869014)|
|:-:|
| Compare Latin letters with Kanji and Kana[^2]|

We can tell `capHeight` may not equal to the text height, but it's (almost?) center aligned to the text, so the calculation above should be accurate enough.

## Usage

Let's see how we can use this `UIFont` extension.

### Auto Layout

We have `firstBaselineAnchor` and `lastBaselineAnchor`, so we can align our view to the center of the first/last line of the text.

``` swift
let offset = label.font
    .bottomOffsetFromBaselineForVerticalCentering(targetHeight: view.height)
view.bottomAnchor
    .constraint(equalTo: label.firstBaselineAnchor, constant: -offset)
    .isActive = true
```

### NSAttributedString

When we add an image to `NSAttributedString`, the image will be layout above the baseline since the origin is on the baseline. However, we might want to center the image horizontally.

``` swift
let textAttachment = NSTextAttachment(image: image)
let imageSize = image.size
let offset = label.font
    .bottomOffsetFromBaselineForVerticalCentering(targetHeight: imageSize.height)
textAttachment.bounds = .init(origin: .init(x: 0, y: offset), size: imageSize)
attributedString.append(NSAttributedString(attachment: textAttachment))
```

## Conclusion

Using the font metrics in `UIFont`, we can cover most of the view-to-text alignment requirements with some calculation like the example above.

Those requirements may be trivial. But, there is no reason to waste the designer's work since `UIFont` has made the implementation easy enough.

## References

[^1]: [Font Handling](https://developer.apple.com/library/archive/documentation/TextFonts/Conceptual/CocoaTextArchitecture/FontHandling/FontHandling.html)

[^2]: [Creating Balanced Typographic Design in Bilingual Websites (Part 1)](https://theplant.jp/articles/creating-balanced-typographic-design-in-bilingual-websites-part-1)
