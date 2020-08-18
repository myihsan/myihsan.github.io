---
layout: post
title: SwiftUIで子ビューのframeを取得
---

UIKitの場合は適切なタイミングで`UIView.frame`から取得できるし、Auto Layoutで制約つけていれば、具体的なframeを取得しなくても、大体な目的は達成できます。SwiftUIの場合は`GeometryReader`で親ビューのframeを取得することは可能ですが、子ビューのframeを取得するに別の手法が必要です。

## 親の方に情報伝達

直接に子ビューのframeを取得することができませんので、子ビュー側で取得してから親に伝達する必要があります。

子の方に伝達するにはenvironmentという仕組みで、逆方向の場合はpreferenceという仕組みがあります。

Preferenceを使うには`PreferenceKey`を実装する必要があります。

```swift
struct FramePreferenceKey: PreferenceKey {

    static var defaultValue: CGRect? = nil

    static func reduce(value: inout CGRect?, nextValue: () -> CGRect?) {
        value = nextValue()
    }
}
```

子ビューに対してmodiferで値を設定します。

```swift
.preference(key: FramePreferenceKey.self, frame)
```

親ビュー側に対してもmodifierで値を取得します。

```swift
.onPreferenceChange(FramePreferenceKey.self) { frame in
    childFrame = frame
}
```

子ビューの方で設定する`frame`はどう取得するかはまだ解決していないのに、ちょっと複雑になりましたよね。私も複雑に感じて、[変わった手法]({% post_url 2020-08-15-getting-subview-frame-in-swiftui-deprecated %})に手を出しました。

しかし、親の方に情報伝達用の`PreferenceKey`便利さはこれからです。

## Frameの取得

`preference(key:value:)`はあくまで一番基本な使い方で、frameを取得するには別の使い方があります。

```swift
.anchorPreference(key: BoundsAnchorPreferenceKey.self, value: .bounds) { $0 }
```

こうやってboundsを[変わった手法]({% post_url 2020-08-15-getting-subview-frame-in-swiftui-deprecated %})を使わなくても、直接にboundsを取得できます。

Anchorという仕組みで取得していますので、`PreferenceKey`の`Value`を変更する必要があります。

```swift
struct BoundsAnchorPreferenceKey: PreferenceKey {

    static var defaultValue: Anchor<CGRect>? = nil

    static func reduce(value: inout CGRect?, nextValue: () -> CGRect?) {
        value = nextValue()
    }
}
```

`anchorPreference(key:value:transform:)`のドキュメントに何も書いてありませんが、取得できる値について[`Anchor.Source`](https://developer.apple.com/documentation/swiftui/anchor/source)で確認できます。

最後、親ビュー側で`GeometryReader`を使い、`GeometryProxy`でframeに変換します。

```swift
GeometryReader { proxy in
    let frame = proxy[anchor]
}
```

## 具体例

![UISegmentedControl](/assets/images/segmented-control.png)

例えば`UISegmentedControl`のように選択された部分が変わった場合、背景をアニメーション付きで移動する部品を作ります。

選択された部分のboundsだけを取得するには、`if`により違う`View`になることを対処する必要があります。`AnyView`はもちろんできるし、`ZStack`で対処する方法が見たことがあるかもしれません。`ZStack`で対処できる根本的な理由はfunction builderのおかげですので、下記のような`extension`でも対処できます。

```swift
extension View {
    @ViewBuilder func modifier<T>(shouldModify: Bool, modifier: (Self) -> T) -> some View where T : View {
        if shouldModify {
            modifier(self)
        } else {
            self
        }
    }
}
```

```swift
.modifier(shouldModify: index == selectedSegmentIndex) { content in
    content
        .anchorPreference(key: BoundsAnchorPreferenceKey.self, value: .bounds) { $0 }
}
```

これで選択された部分のboundsだけを取得できるようになりました。

次は選択された部分の背景を作ります。

まさにこのような需要のために設計されたようなmodiferがあります。

```swift
.backgroundPreferenceValue(BoundsAnchorPreferenceKey.self) { anchor in
    anchor.map { anchor in
        GeometryReader { proxy in
            buildSelectedBackground(by: proxy[anchor])
        }
    }
}
```

`Anchor<CGRect>?`の`nil`に対してまた`AnyView(EmptyView())`とかで対処する必要があると思いきや、`Optional.map(_:)`で普通に対処できます。

かつ`animation(_:)`つけている場合でも、`nil`のおかげで、背景は初期状態は`.zero`から取得できたframeになるのではなく、背景なしから背景ありになりますので、[変わった手法]({% post_url 2020-08-15-getting-subview-frame-in-swiftui-deprecated %})の方にある不要なアニメーションがありません。

## まとめ

1. `anchorPreference(key:value:transform:)`でboundsを取得
2. `GeometryReader`を使い、`GeometryProxy`でframeに変換
   - 優先に`backgroundPreferenceValue(_:_:)`や`overlayPreferenceValue(_:_:)`を検討し、仕方がない場合`onPreferenceChange(_:perform:)`を使う

## 参考

- [Conditionally apply modifier in SwiftUI](https://forums.swift.org/t/conditionally-apply-modifier-in-swiftui/32815/19)

## 感謝

[変わった手法]({% post_url 2020-08-15-getting-subview-frame-in-swiftui-deprecated %})に対して違和感を感じていましたが、よりいいやり方がわからなくて、記事にしましたが、[auramagi](https://github.com/auramagi)さんにこの標準的な手法を教えていただきました。
