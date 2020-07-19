---
layout: post
title: DragGesture
---

ドラッグジェスチャーはiOSにおいてタップジェスチャー以外一番使われるジェスチャーで、SwiftUIにおいて`DragGesture`で実装されます。

今までの`UIPanGestureRecognizer`と同じくらいなことができるかを見てみましょう。

## イニシャライザ

```swift
init(minimumDistance: CGFloat = 10, coordinateSpace: CoordinateSpace = .local)
```

`minimumDistance`以下であれば反応しないように設定することが可能で、座標空間を指定することも可能です。

座標空間を指定することで、これから紹介する`DragGesture.Value`にある値が決められます。`UIPanGestureRecognizer`の`translation(in:)`のように都度変更できません。

## ステータス

### 経過

```swift
func onChanged(_ action: @escaping (DragGesture.Value) -> Void) -> _ChangedGesture<DragGesture>
```

`UIGestureRecognizer.State.changed`と同じタイミングで、ジェスチャーの経過が変化された場合実行されます。

関心な`DragGesture.Value`を見てみましょう。

- `translation`
- `startLocation`
- `location`

`UIPanGestureRecognizer`の`translation(in:)`より`startLocation`と`location`が増えました。三つの中二つ分かればもう一つを計算することが可能ですので、実質一つしか増えていません。

- `time`

`UIPanGestureRecognizer`の`velocity(in:)`がなくて、`translation`と`time`で計算できると思いきや、現時点`time`はジェスチャーが始めてからの時間ではないようでスピードの計算ができません。

- `predictedEndLocation`
- `predictedEndTranslation`

とはいえ、現在の位置とスピードで予測された終了時の`location`と`translation`がありますので、この二つの値を使っていれば`UIPanGestureRecognizer`の`velocity(in:)`を使う時と似たようなことができます。

### 終了

```swift
func onEnded(_ action: @escaping (DragGesture.Value) -> Void) -> _EndedGesture<DragGesture>
```

`UIGestureRecognizer.State.ended`と同じタイミングで、ジェスチャーが終了した時に実行されます。

### 引き換えがないステータス

- `UIGestureRecognizer.State.began`

`onChanged(_:)`が初めて呼んだ時と同じタイミングですので、`onBegan(_:)`がなくても困りません。

- `UIGestureRecognizer.State.possible`

残念ですが、現時点SwiftUIにおいてこの状態の検知ができません。よく使われるステータスではありませんので、そこまで困りませんでしょう。

- `UIGestureRecognizer.State.cancelled`
- `UIGestureRecognizer.State.failed`

`UIGestureRecognizer.State.ended`と同じく`onEnded(_:)`になると期待していましたが、この二つのステータスになる場合`onEnded(_:)`実行されません。

ジェスチャーが終了された時のみ何かをする場合ほぼ影響がありませんが、ジェスチャーの途中でステータスを記録したり、レイアウトを変更したりする場合、キャンセルや失敗した時元に戻す必要があるのに通知されないのは困ります。

この問題について一応[Developer Forums](https://developer.apple.com/forums/thread/654620)に質問を投げましたが、おそらくまだ実装されていないと思います。

現時点とれる対策として下記のファンクションを使います。

```swift
func updating<State>(_ state: GestureState<State>, body: @escaping (DragGesture.Value, inout State, inout Transaction) -> Void) -> GestureStateGesture<DragGesture, State>
```

公式の例を見てみましょう。

```swift
struct CounterView: View {
    @GestureState var isDetectingLongPress = false

    var body: some View {
        let press = LongPressGesture(minimumDuration: 1)
            .updating($isDetectingLongPress) { currentState, gestureState, transaction in
                gestureState = currentState
            }

        return Circle()
            .fill(isDetectingLongPress ? Color.yellow : Color.green)
            .frame(width: 100, height: 100, alignment: .center)
            .gesture(press)
    }
}
```

一見`GestureState`に値を保持しているだけで、`onChanged(_:)`で自前でもできることみたいですが、関心なのは`GestureState`です。

> A property wrapper type that updates a property while the user performs a gesture and resets the property back to its initial state when the gesture ends.

こちらも「ends」に書いてありますが、私がテストした限りにはキャンセルや失敗した時も初期値に戻ってくれます。

しかしここで一つ問題が出てきます。

```swift
func updating<State>(_ state: GestureState<State>, body: @escaping (DragGesture.Value, inout State, inout Transaction) -> Void) -> GestureStateGesture<DragGesture, State>
```

パラメーターのクロージャを注目してください。`onChanged(_:)`と`onEnded(_:)`の`action`とは違い、`body`です。名前通りにアクションのクロージャより`View`の`body`のようなもので、`State`を変更すると下記のワーニングが出てきます。

> Modifying state during view update, this will cause undefined behavior.

できるのはクロージャにある`state`を経由し、`GestureState`を変更することです。`withAnimation(_:_:)`も効きません。

このせいで、ジェスチャーの途中でステータスを記録したり、レイアウトを変更したりする場合、アニメーション付きで元に戻すためにビュー側に`animation(_:)`でアニメションを常に発生するように設定する必要があります。

しかしこれでドラッグした距離を直接ビューに反映することができなくなり、必ずアニメション経由で反映するようになりました。キャンセルや失敗した時が検知できないよりはましですが、やはり不自然です。

根本的に解決できませんが、一応スピードやレスポンスが早いアニメーションであれば違和感なくなると思います。例えば：

```swift
static func interactiveSpring(response: Double = 0.15, dampingFraction: Double = 0.86, blendDuration: Double = 0.25) -> Animation
```

とはいえ`animation(_:)`つけるといろいろ不便になりますので、これはあくまで一時的な対応策で、APIの更新を待ちましょう。

## ビューに紐付け

```swift
content
    .gesture(DragGesture())
```

SwiftUIによくあるお馴染みのModifierだけでできて、非常に簡単です。

## まとめ

やはりUIKitと比べると結構な不自由がありますね。

ジェスチャーの検知ぐらいは大丈夫ですが、ジェスチャーの途中でステータスを記録したり、レイアウトを変更したい場合、ドラッグジェスチャーを採用するかどうか慎重に考えましょう。

## 参考

- [DragGesture](https://developer.apple.com/documentation/swiftui/draggesture)
- [Adding Interactivity with Gestures](https://developer.apple.com/documentation/swiftui/adding-interactivity-with-gestures)
