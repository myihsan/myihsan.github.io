---
layout: post
title: "エンジニア向けデザイン: SwiftUI"
---

先々週、一つセルのデザインのため、どれだけやらなければならないことがあるかを見てみました。エンジニアに渡す前にも伝えるため、各パターンの画像を準備する必要で、大変でしょう。では直接に考えたレイアウトを説明できる形があれば使いたいですよね。開発用のSwiftUIですが、レイアウトを説明するにはぴったりです。

SwiftUIでもう一度[セルをデザイン]({% post_url 2019-11-03-viewmodel-a-superhero-belongs-to-androidx %})してみましょう。

## 準備

まず[Swift Playgrounds](https://apps.apple.com/jp/app/swift-playgrounds/id1496833156?l=en&mt=12)または[Xcode](https://apps.apple.com/jp/app/xcode/id497799835?l=en&mt=12)をApp Storeからダウンロードします。

- Swift Playgrounds: File > New Blank Playground
- Xcode: File > New > Playground... > Blank

そして下記のコードをコピペします。

```swift
import SwiftUI
import PlaygroundSupport

PlaygroundPage.current.setLiveView(Cell())
```

`Cell`を作成してから右下（Xcodeは左下）の▶︎ボタンを押せば、デザインをプレビューできます。

`Cell`のレイアウトを説明する前に、まずタイトルとラベルの説明をします。

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

これで、タイトルとラベルのフォントと色を説明しました。

## セルのデザイン

`Cell`のデザインを始めましょう！

| ![一般的なデザイン](/assets/images/common-cell-design.png) |
|:-:|
| 一般的なデザイン |

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

ちょっと複雑ですよね。部分的に見てみましょう。

```swift
.padding(.horizontal, 16)
.frame(maxWidth: .infinity, minHeight: 44)
.background(Color.white)
```

この部分は簡単で、`Text`のフォントと色を説明したように、`ZStack`の左右のスペース、最小高さ、背景色を説明しました。

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

タイトルとラベル`Spacer`と一緒にを`HStack`（水平レイアウト）を入れました。`Spacer`は残ったスペースを全部占めてくれて、先頭揃えと後尾揃えができました。

最後、この二つの`HStack`を`ZStack`に入れて、タイトルとラベルが重ねられるようにしました。

なぜ直接に一つの`HStack`を入れないのと思いますよね。これは一般的なデザインツールの挙動をマネしているだけです。デザインツールに置いてデフォルトの挙動だけど、逆に煩雑になります。

SwiftUIでデザインすると、このようになります。

```swift
HStack {
    CellTitle(text: "Title")
    Spacer()
    CellLabel(text: "Label")
}
```

タイトルとラベルの背景色をつけましょう。

| ![コンテンツエリア](/assets/images/visible-content-area.png) |
|:-:|
| 見えるコンテンツエリア |

```swift
HStack {
    CellTitle(text: "Title")
        .background(Color.green)
    Spacer()
    CellLabel(text: "Label")
        .background(Color.yellow)
}
```

このステップが必要ではありませんが、SwiftUIを慣れる前に背景色がコンテンツエリアを明記的にしました。

次は最大幅を定義しましょう。

| ![明確なコンテンツエリア](/assets/images/explicit-content-area.png) |
|:-:|
| 明確なコンテンツエリア |

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

今度は最大幅を説明したことで、タイトルとラベルのコンテンツエリアを明確にしました。ラベルは固定な最大幅があり、タイトルが無限で残ったスペースを占めます。`Space`の最小幅を説明することで、タイトルとラベルの間隔を説明しました。

一行を超えるタイトルを切り捨てる場合：

| ![切り捨てたテキスト](/assets/images/truncating-text.png) |
|:-:|
| 切り捨てたテキスト |

```swift
CellTitle(text: "A Long Enough Title That Will Take Two Lines")
    .lineLimit(1)
    ...
```

This is the default behavior of SwiftUI, so all we need to do is to limit the line of the title.
これはSwiftUIのデフォルトの挙動で、必要な説明はテキストを一行に制限することだけです。

幅によりフォントサイズを調整する場合：

| ![フォント調整したテキスト](/assets/images/autosizing-text.png) |
|:-:|
| フォント調整したテキスト |

```swift
CellTitle(text: "A Long Enough Title That Will Take Two Lines")
    .lineLimit(1)
    .minimumScaleFactor(0.5)
    ...
```

行数制限した上で、最小縮小倍率を説明すればできます。

これでタイトルのフォントがラベルより小さくなり、中央揃えからベースラン揃えに変更した方が揃えた感じがします。

```swift
HStack(alignment: .lastTextBaseline) {
    ...
}
```

タイトルを改行できるデザインにする場合：

| ![高さ可変コンテンツエリア](/assets/images/extendable-content-area.png) |
|:-:|
| 高さ可変コンテンツエリア |

```swift
HStack {
    ...
}
.padding(.vertical, 10)
...
```

`lineLimit(_:)`をなくして、縦のスペースを説明すればできます。

## 結論

SwiftUIのおかげで、コードでデザインするのもそんなに難しくないでしょう？ここまでやったのはただ英語でデザインを説明しただけです。

[ここで](https://github.com/myihsan/ProgrammerOrientedUIDesign)そのまま実行できるソースコードがダウンロードできます.

SwiftUI以外にも、いろんなツールがデザインを説明できます。例えばFlutterとJetpack Compose。どちらでも似たような文法があり、エンジニア側使っているツールによって説明のツールも合わせてあげましょう！
