---
layout: post
title: Timerを使うコードのユニットテスト
---

他のタイプと違い、インスタンスを作る時イニシャライザよりクラスメソッド（`scheduledTimer`）の方がよく使われ、作られたインスタンスから`repeats`であるかどうか確認できませんので、テストにはちょっと変わった方法が必要です。

間隔ごとに`Timer`が発火することをテストしにくいし、厳格にいえば不可能（永遠にテストが続ける）ですので、テストすべきなのは`Timer`はどういう引数で作られ、`fire()`と`invalidate()`が実行されることです。

そのため`Timer`をモックする必要があります。Swiftにおいてモックといえば、サブクラスかプロトコルかになります。

## サブクラス

ありがたいことに`scheduledTimer`が全てクラスメソッドでオーバーライドすることが可能ですし、`Timer`がclass clusters[^1]でイニシャライザをオーバーライドするのはできないですが、`NSObject`のおかげで引数なしのイニシャライザがありますので、イニシャライザまでモックする必要がありません。

モッククラスを作るには大体下記の形になります。

```swift
class MockTimer: Timer {
    override class func scheduledTimer(withTimeInterval interval: TimeInterval, repeats: Bool, block: @escaping (Timer) -> Void) -> Timer {
        MockTimer()
    }

    var fireCalled = false
    override func fire() {
        fireCalled = true
    }

    var invalidateCalled = false
    override func invalidate() {
        invalidateCalled = true
    }
}
```

必要に応じて他の`scheduledTimer`をオーバーライドしたり、`fire()`の実行回数をカウントしたりします。しかし`Timer`がclass clusters[^1]のせいで、オーバーライドしていないものは全部使えません。

Timerを使う側にインジェクトできるようにします。

```swift
class SomeClass {
    private let timerType: Timer.Type
    init(timerType: Timer.Type = Timer.self) {
        self.timerType = timerType
    }
    ...
    func someMethod() {
        let timer = timerType.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in
            ...
        }
        time.fire()
    }
    ...
}
```

普通にはこれでテスト書けるようになりますが、`Timer`のインスタンスが都度作られますので、それをキャッチしてテストする必要があります。

クラスメソッドですので、スタティックメンバーでキャッチしましょう。

```swift
class MockTimer {
    static var scheduledTimer: MockTimer?
    static var timerScheduledBy: (interval: TimeInterval, repeats: Bool, block: (Timer) -> Void)?

    override class func scheduledTimer(withTimeInterval interval: TimeInterval, repeats: Bool, block: @escaping (Timer) -> Void) -> Timer {
        let scheduledTimer = MockTimer()
        self.scheduledTimer = scheduledTimer
        self.timerScheduledBy = (interval, repeats, block)
        return scheduledTimer
    }
    ...
}
```

もっとリアルタイムにチャックする必要があれば、`MockTimer?`などを`((MockTimer) -> Void)?`などに変更すれば十分だと思います。

テストの一例:

```swift
override func tearDown(){
    MockTimer.scheduledTimer = nil
    MockTimer.timerScheduledBy = nil
}

func testSomeMethod() {
    let someClass = SomeClass(timerType: MockTimer.self)
    someClass.someMethod()

    let timer = MockTimer.scheduledTimer
    let (interval, repeats, _) = MockTimer.timerScheduledBy
    XCTAssertEqual(interval, 1)
    XCTAssertTrue(repeats)
    XCTAssertTrue(timer.fireCalled)
}
```

## プロトコル

使う予定ある`Timer`のプロパティやメソッドを参照し、プロトコルを作ります。

```swift
protocol TimerProtocol {
    func fire()
    func invalidate()
    ...
}

extension Timer: TimerProtocol {}
```

`scheduledTimer`の方も同じようにしてみます。

```swift
protocol TimerScheduler {
    static func scheduledTimer(withTimeInterval interval: TimeInterval, repeats: Bool, block: @escaping (Self) -> Void) -> Self
}
```

残念ながらクラスメソッドは`Self`の制約を満たすことができません。`Timer`のままの場合`TimerProtocol`を返すことができなくなります。

仕方ありませんので、ラッピングしてあげましょう。

```swift
protocol TimerScheduler {
    static func scheduledTimer(withTimeInterval interval: TimeInterval, repeats: Bool, block: @escaping (TimerProtocol) -> Void) -> TimerProtocol
}

extension Timer: TimerScheduler {
    static func scheduledTimer(withTimeInterval interval: TimeInterval, repeats: Bool, block: @escaping (TimerProtocol) -> Void) -> TimerProtocol {
        let timer: Timer = scheduledTimer(withTimeInterval: interval, repeats: repeats, block: block)
        return timer
    }
}
```

この上でモックのタイプを作れば[^2]テスト可能になります。テストコードはサブクラスの方とあまり変わりませんので省略します。

## モックの問題

上記の方法で確かに`Timer`を使うコードがテストできるようになりますが、場合により一つの前提が必要です。それは`Timer`内部の動きが把握できていることです。

例えばテストするコードは`isValid`により違うことをする場合、`invalidate()`が実行されましたら、`isValid`の値を`false`にする必要があります。そして例え把握していても、その通りに実装できていなければ意味がありません。また、ほぼあり得ない話ですが、アップルが`Timer`の実装が変えたら、テストは無効になります。

そのため、実体を使うようにテストするのもありかと思います。

```swift
var blockCalled = false
let timer = Timer(timeInterval: 1, repeats: true, block: { _ in
    blockCalled = true
})
```

このような`Timer`をモックした`scheduledTimer`に返してあげることで、`Timer`のままでテストできます。`RunLoop`にスケジュールしていませんので、`fire()`しない限りには`block`は実行されません。

とはいえ`Timer`の仕様上こういうことができるだけで、普通の場合モックでテストすれば十分だと思います。

## まとめ

ちょっと変わった方法が必要とはいえ、モックの基本手法ーサブクラスとプロトコルで解決できます。

モックの問題について、`Timer`はそこまで複雑ではありまんので、問題ないでしょうが、`Timer`のようにあるメソッド（`invalidate()`）他のアウトプット（`isValid`）に変化があり、かつロジックは複雑な場合、モックしない方が無難かもしれません。

***

[^1]: [Class Clusters](https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaEncyclopedia/ClassClusters/ClassClusters.html)

[^2]: 自分で作るより[Mockingbird](https://github.com/birdrides/mockingbird)や[Cuckoo](https://github.com/Brightify/Cuckoo)のようなモックフレームワークに任せましょう。（Cuckooは`static`をサポートしていないため、ラッピングの形を変える必要がある）
