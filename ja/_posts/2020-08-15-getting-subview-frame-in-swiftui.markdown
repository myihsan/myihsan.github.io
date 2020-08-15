---
layout: post
title: SwiftUIで子ビューのフレームを取得
---

UIKitの場合は適切なタイミングで`UIView.frame`から取得できるし、Auto Layoutで制約つけていれば、具体的なフレームを取得しなくても、大体な目的は達成できます。SwiftUIの場合は`GeometryReader`で親ビューのフレームを取得することは可能ですが、子ビューのフレームを取得するにはちょっと変わった手法が必要です。

ちょっと変わった手法ですので、より普通な手法があればぜひ指摘お願いします。

## 原理

> `GeometryReader`で親ビューのフレームを取得することは可能

この前提から考えると、子ビューを親ビューにすればできます。問題は`GeometryReader`が子ビューに入って、フレームを親に伝えるため`@Binding`を使う必要があり、子ビューが複雑になります。

子ビューを調整したいですが、子ビューそのものを変更したくない場合、modifierの出番です。`overlay(_:alignment:)`や`background(_:alignment:)`を使えば問題が解決できます。

```swift
.overlay(
    GeometryReader { proxy in
        Color.clear
    }
)
```

変わった手法で、`EmptyView`を使いたいですが、`EmptyView`は特殊[^1]で使えません。

## 運用

`GeometryReader`を使い、`GeometryProxy`からフレームを取得することが可能になりましたが、どう使うのは次の問題になります。

まずはフレームの座標系を指定する必要があります。UIViewの場合は`convert(_:from:)`で指定できますが、SwiftUIはテキストで指定します。

```swift
VStack {
    ...
    subview
        .overlay(
            GeometryReader { proxy -> Color in
                let frame = proxy.frame(in: .named("parent"))
                return .clear
            }
        )
    ...
}
.coordinateSpace(name: "parent")
```

具体的なフレームを取得できましたので、`@State`にセットすれば終わりですが、`GeometryReader`は`View`ですので、直接にセットすると下記に警告が出て、動きもおかしくなります。

> Modifying state during view update, this will cause undefined behavior.

SwiftUIの都合で、回避しなければなりませんが、`DispatchQueue.main.async(execute:)`で簡単に回避できます。

```swift
.overlay(
    GeometryReader { proxy -> Color in
        DispatchQueue.main.async {
            frame = proxy.frame(in: .named("container"))
        }
        return .clear
    }
)
```

もっと改善する余地[^3]はありますが、これで子ビューのフレームを取得し、利用できるようになります。

## 具体例

![UISegmentedControl](/assets/images/segmented-control.png)

例えば`UISegmentedControl`のように選択された部分が変わった場合、背景をアニメーション付きで移動する部品を作ります。

```swift
private var background: some View {
    ...
    background
        .frame(width: selectedFrame.width, height: selectedFrame.height)
        .offset(x: selectedFrame.minX, y: selectedFrame.minY)
    return background
}


var body: some View {
    HStack {
        ...
    }
    .background(background, alignment: .topLeading)
}
```

上記の方法で1個目の部分のフレームを取得して、このように背景をレイアウトすればできます。

別の部分を選択された時、インデックスを変えて同じことをやればできますが、そもそも選択されたときは描画する時ではなく、タップされた時ですのでが`DispatchQueue.main.async(execute:)`使う必要がありません。

```swift
.overlay(
    GeometryReader { proxy in
        Button(action: {
            withAnimation {
                selectedFrame = proxy.frame(in: .named("parent"))
            }
        }, label: {
            Color.clear
        })
    }
)
```

`overlay(_:alignment:)`を使い、フレームを取得する手法は変わりませんが、`DispatchQueue.main.async(execute:)`がなくなったし、描画時と変更時が分けて、変更時だけアニメーションをつけることができます。

普通の場合これで終わりますが、もしもっと親の方が`animation(_:)`つけている場合、`withAnimation(_:_:)`でコントロールできなくなります。そのためもうちょっと対応する必要があります。

```swift
@State var selectedFrame: CGRect = .zero {
    didSet {
        if oldValue != .zero {
            selectionChanged = true
        }
    }
}
@State var selectionChanged = false
```

この上、`animation(_:)`でアニメーションを上書きします。

```swift
private var background: some View {
    ...
    background
        .frame(width: selectedFrame.width, height: selectedFrame.height)
        .offset(x: selectedFrame.minX, y: selectedFrame.minY)
        .animation(selectionChanged ? .easeOut : nil)
    return background
}
```

これで初回表示がアニメーションなしになります。しかし、一回選択すると、向きの変更が発生したてきもアニメーション付きになってしまいます。幸いなことに`DispatchQueue.main.async(execute:)`で解決できます。

```swift
private var background: some View {
    ...
    background
        .frame(width: selectedFrame.width, height: selectedFrame.height)
        .offset(x: selectedFrame.minX, y: selectedFrame.minY)
        .animation(selectionChanged ? .easeOut : nil)
    DispatchQueue.main.async {
        selectionChanged = false
    }
    return background
}
```

こうやって、アニメーションを先に始めさせて、それからアニメーションを`nil`にすることで、選択した時だけアニメーションをつけることができます。

とはいえ、このアプローチはアニメーションを`nil`にしても、発生しているアニメーションに影響したいという前提に依存していますので、今後できなくなる可能性があります。ですので、`animation(_:)`をつける必要があるような仕方がない場合以外はこのアプローチを避けましょう。

## まとめ

1. `overlay(_:alignment:)`や`background(_:alignment:)`でフレームを取得
2. `DispatchQueue.main.async`で`@State`に反映

## 参考

- [ios - How to make view the size of another view in SwiftUI - Stack Overflow](https://stackoverflow.com/questions/56505043/how-to-make-view-the-size-of-another-view-in-swiftui)

***

[^1]: 個人的な推測ですが、`EmptyView()`はただプレースホルダー的なもので、最適化の段階や実際のレイアウトするとき取り除かれます。

[^2]: 同じ値でなければ止まりませんので、同じ値を二回取得し、セットしたまで止まりません。

[^3]: `AnyView`でラップしてあげて、`EmptyView`を返すことや条件式により違うレイアウトするmodiferを作ることで、初回に以外に`GeometryReader`をなくすこともできます。