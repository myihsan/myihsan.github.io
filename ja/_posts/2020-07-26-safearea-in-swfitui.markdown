---
layout: post
title: SwiftUIのSafe Area対応
---

Auto Layoutにおいて制約を`safeAreaLayoutGuide`に設定するかどうかの使い分けでSafe Area対応が簡単にできますが、SwiftUIにおいてレイアウトの仕方が完全に変わったため、似たような対応ができなくなりました。

API自体もまだまだ更新されつつけていますが、現時点までのSafe Area対応方法を見てみましょう。

## UIKit的な考え方

`safeAreaLayoutGuide`はありませんが、`GeometryReader`によって`safeAreaInsets`を取得することができます。

ではUIKitのように、親の方Safe Areaを無視してレイアウトし、子の方が`safeAreaInsets`と同じスペースを空けばできます。

```swift
GeometryReader { proxy.safeAreaInsets
    content
        .padding(proxy.safeAreaInsets)
}
.edgesIgnoringSafeArea(.all)
```

これができれば、今までの考え方で対応ですますが、iOS 14からSafe Areaを無視したら、`safeAreaInsets`がの値が0になりますので、できなくなっています。

## 食み出し

スペースを空ける方法はできないなら、食み出すようにしてみよう。

```swift
GeometryReader { proxy.safeAreaInsets
    content
        .padding(.bottom, proxy.safeAreaInsets.bottom)
        .background(backgroundColor)
        .padding(.bottom, -proxy.safeAreaInsets.bottom)
}
```

普通にできますが、各方向をマイナスにする必要があるため、ちょっと手間がかかります。

## 自然的な対応

上記の方法はできるとはいえ、違和感があるますよね。

標準としてドキュメントに書いているわけではありませんが、もっと自然的な対応方法があります。

```swift
content
    .background(backgroundColor.edgesIgnoringSafeArea(.all))
```

直接に`content`の方につけても無視できませんが、`background(_:alignment:)`経由であればできます。よく考えてみれば、おおよその場合は背景を食み出せば十分で、そのために設計されているかもしれません。

似たような`overlay(_:alignment:)`でもできます。

## キーボード

皆さんもキーボードが入力欄などを隠さないようにいろいろ対応してきたと思いますが、このような対応はついに楽になってきました。

なぜかというとiOS 14 beta 3から、キーボードの出入りがSafe Areaを影響するようになりました。そのため[`ignoresSafeArea(_:edges:)`](https://developer.apple.com/documentation/swiftui/vstack/ignoressafearea(_:edges:))も追加されました。

Safe Areaを無視していない限りには対応不要になります。今までのようにキーボードの影響がないSafe Areaが必要な場合、下記のようにできます。

```swift
content
    .ignoresSafeArea(.keyboard)
```

## まとめ

- `GeometryReader`を使い、マイナスの`padding`で食み出す
- `background(_:alignment:)`経由で`ignoresSafeArea(_:edges:)`

前者の方はちょっとトリッキーですので、特別な必要がなければ後者で対応しましょう。