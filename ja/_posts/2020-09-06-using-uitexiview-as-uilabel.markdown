---
layout: post
title: UITextViewをUILabelとして使用
---

テキストを選択できるようにしたら、リンクが開けるようにしたり、スクロールできるようにしたりするため、`UILabel`をわかりに`UITextView`を使いがちですが、いったい`UILabel`と完全に同じようになれるでしょうか？

## 操作を無効

```swift
textView.isEditable = false
// 必要に応じて
textView.isSelectable = false
textView.isScrollEnabled = false
```

## 周囲のスペースを除去

```swift
textView.textContainerInset = .zero
textView.textContainer.lineFragmentPadding = 0
```

## attributedText

基本的に上記の対応は十分ですが、`text`ではなく、`attributedText`を使う場合いろいろな問題が出ます。

### フォント

`UILabel`の場合、前もって`font`を設定していれば、後で設定する`attributedText`にも反映できます。しかし`UITextView`では前もって`font`を設定することができません。

とはいえ`attributedText`を設定した度に、`font`を設定すること、または`NSAttributedString`にフォントを指定することで問題が解決できます。

ここにワナがあります。`attributedText`を設定した度に、`font`がデフォルトに戻されるように見え、デフォルトフォントで良いの場合は`font`を設定しなくてもいいと思いますよね。そうでもありません。

| ![`text`｜`attributedText`](/assets/images/uitextview-text-vs-attributed-text.png) |
|:-:|
| `text`｜`attributedText` |

サイズが14ptから12ptになっただけではなく、フォントもなぜかHelveticaになりました。

### 下線

フォントをちゃんと設定すればこれ以上の問題がないと思いきや、下線の位置が違います。

| ![`UILabel`｜`UITextView`](/assets/images/uilabel-underline-vs-uitextview-underline.png) |
|:-:|
| `UILabel`｜`UITextView` |

フォントは完全に一緒ですが、下線だけ違うということは、簡単に対応できる方法がないはずです。

## ヒラギノ

フォントをヒラギノに設定することは[おすすめしない](https://qiita.com/yusuga/items/2be8c55ca561bba44702)のですが、もし設定したらどうなるかを見てみましょう。

| ![`UILabel`｜`UITextView`](/assets/images/uilabel-underline-with-hiragino-sans-vs-uitextview-underline-with-hiragino-sans.png) |
|:-:|
| `UILabel`｜`UITextView` |

下線の位置の違いがもって大きいし、下に大きなスペースがあります。

測ってみたら、`UIFont.leading`が使われているように見えます。システムフォントの場合、このスペースがないのはシステムフォントの`leading`が0だからかもしれません。

ちなみにフォントを設定しないで`attributedText`を設定し、Helveticaになる場合はシステムフォントのような[特別な調整]({% post_url 2020-09-06-using-uitexiview-as-uilabel %})がなくて、直接にHiragino Sansに変更されて、画像のように文字と下線の距離は結構あるため、下線が外部に出て、表示できなくなります。

## 可能な原因

`UILabel`は静的な部品ですが、`UITextView`は入力のために作られた部品で、入力しながら行の高さや下線の位置がわからない（Xcodeができていない😅）ようにするため、`UILabel`よりいろんな工夫をしているはずで、それが原因だと私が思います。

## まとめ

`UITextView`を`UILabel`として使うことは可能ですが、細かいところ（特に`attributedText`を設定する場合）に違いが出ます。とはいえ大きな違いではないので、気にしなくても良いかと思います。

気にする場合は`UILabel`を使って、必要な機能を別の方向で実装しましょう。

- 選択可能：無理？
- リンクが開ける：`UITapGestureRecognizer`
- スクロール可能：`UIScrollView`