---
layout: post
title: 数値型のラッパー
---

Swiftにはいろんな数値型があり、大体の場合は十分です。時にはカスタムな数値型が必要ですが、ラッパーを作っていれば大半な需要が満たせます。

一つよくあるパターンは文字を数値としてデコードする必要がある場合かと思います。カスタムの型にすることで、デコードするロジックを一箇所にまとめます。

```swift
struct MyDecimal: Decodable {

    let decimal: Decimal

    init(from decoder: Decoder) {
        // Custom decoding
    }
}
```

しかしこれで計算や比較したい場合、`decimal`でいちいち取り出さないといけません。やはり`Decimal`のように使いたいですよね。

昔々の記憶により、プロトコルになっていなくて、どれが必要かの一覧もありませんでした（私知らないだけかも）ので、忘れがちでしたが、Xcode 10.2以降であれば、大体必要な機能がプロトコルになりましたので、参考してみましょう。

`Decimal`は下記のプロトコルを実装しています。

- Comparable
- CustomStringConvertible
- Decodable
- Encodable
- ExpressibleByFloatLiteral
- ExpressibleByIntegerLiteral
- Hashable
- SignedNumeric
  - Numeric
    - AdditiveArithmetic
      - Equatable
    - ExpressibleByIntegerLiteral
- Strideable

これで一覧がありますが、結構の数量ですね。とはいえ、デフォルト実装がありますし、デフォルト実装がないところは`Decimal`の実装をそのまま使えますので、ラッパーを作るだけでは想像より簡単です。

## Comparable・Equatable（Hashable）

```swift
extension MyDecimal: Comparable, Equatable {}
```

`Decimal`は`Comparable`と`Equatable`ですので、デフォルト実装でいけます。

## CustomStringConvertible

```swift
"\(myDecimal)"
String(describing: myDecimal)
myDecimal.description
```

などの場合に使われて、デフォルト実装は`MyDecimal(decimal: 0)`のようになりますので、実装する必要があります。

```swift
extension MyDecimal: CustomStringConvertible {

    var description: String {
        decimal.description
    }
}
```

実装とはいえ、`Decimal`の実装を転送するだけです。

## Codable

必要がなければ、実装しなくても数値型にとして使うのは十分です。また、`Decimal`は`Codable`ですので、カスタムエンコードとデコードロジックがなければ、デフォルト実装で済みます。

今回の例は`Decodable`が実装済みですので、真逆に`Encodable`を実装すれば良いでしょう。

## ExpressibleByIntegerLiteral・ExpressibleByFloatLiteral

リテラルを直接に代入、比較もしくは計算において必要です。

```swift
extension MyDecimal: ExpressibleByIntegerLiteral {

    init(integerLiteral value: Int) {
        self.init(decimal: .init(integerLiteral: value))
    }
}

extension MyDecimal: ExpressibleByFloatLiteral {

    init(floatLiteral value: Double) {
        self.init(decimal: .init(floatLiteral: value))
    }
}
```

デフォルト実装はもちろんありませんが、実装は同じ`Decimal`の実装に転送するだけです。

## AdditiveArithmetic

一つのプロパティーと五つのメソッドで`Decimal`の実装に転送するだけでもちょっと手間かかるなと思いきや、デフォルト実装がありますので、必要なのは最小限の二つのメソッドです。

```swift
extension MyDecimal: AdditiveArithmetic {

    static func + (lhs: MyDecimal, rhs: MyDecimal) -> MyDecimal {
        .init(decimal: lhs.decimal + rhs.decimal)
    }

    static func - (lhs: MyDecimal, rhs: MyDecimal) -> MyDecimal {
        .init(decimal: lhs.decimal - rhs.decimal)
    }
}
```

`Decimal`の状態で同じ計算すれば、簡単にできます。

## SignedNumeric

`AdditiveArithmetic`と似たようなものですが、掛け算の特殊性により、`magnitude`と`init(exactly:)`を実装する必要があります。それにしても同じく転送すれば十分です。またデフォルト実装がありますので、`Numeric`を実装すれば十分です。

```swift
extension MyDecimal: SignedNumeric {

    var magnitude: CommaGroupedDecimal {
        .init(decimal: decimal.magnitude)
    }

    init?<T>(exactly source: T) where T : BinaryInteger {
        guard let source = Decimal(exactly: source) else {
            return nil
        }

        self.init(decimal: source)
    }

    static func * (lhs: MyDecimal, rhs: MyDecimal) -> MyDecimal {
        .init(decimal: lhs.decimal * rhs.decimal)
    }

    static func *= (lhs: inout MyDecimal, rhs: MyDecimal) {
        lhs = .init(decimal: lhs.decimal * rhs.decimal)
    }
}
```

Xcode 9.0に実装されたから放置されたせいかもしれませんが、`*=(_:_:)`のデフォルト実装はありません。

## Strideable

`Codable`と同じく、必要なければ実装がは要りませんし、必要がある場合、同じく転送すれば十分です。

```swift
extension MyDecimal: Strideable {

    func advanced(by n: MyDecimal) -> MyDecimal {
        .init(decimal: decimal.advanced(by: n.decimal))
    }

    func distance(to other: MyDecimal) -> MyDecimal {
        .init(decimal: decimal.distance(to: other.decimal))
    }
}
```

しかし、`Decimal`の`Equatable`は`Strideable`のデフォルト実装を採用していなくて、このままではイコールのチェックする時、`distance(to:)`に入り実行時エラーになります。下記のようにしてあげる必要があります。

```swift
extension MyDecimal: Equatable {

    static func == (lhs: MyDecimal, rhs: MyDecimal) -> Bool {
        lhs.value == rhs.value
    }
}
```

## /(\_:\_:)・/=(\_:\_:)

なぜかよく使われる機能の中に唯一プロトコル化されていない機能があります。忘れないように実装してあげましょう。

```swift
extension MyDecimal {

    static func / (lhs: MyDecimal, rhs: MyDecimal) -> MyDecimal {
        .init(decimal: lhs.decimal / rhs.decimal)
    }

    static func /= (lhs: inout MyDecimal, rhs: MyDecimal) {
        lhs = .init(decimal: lhs.decimal / rhs.decimal)
    }
}
```

ここまで実装してあげれば、`MyDecimal`を数値型として使えるようにになりました。これ以上の需要があるのはもちろんありえますが、実際に必要になってからまだ実装すればいいかと思います。

## まとめ

別の型の機能はそこまでプロトコルとしてまとめていませんので、違うかもしれませんが、数値型のラッパーを作るとき、その数値型が実装しているプロトコルは結構参考になります。
