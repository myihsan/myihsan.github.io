---
layout: post
title: SwiftUIでドラッグ＆ドロップ
---

UIKitでドラッグ＆ドロップいろんなことができていますが、SwiftUIではどこまでしょうか？

一緒に見ていきましょう。

## ドラック

```swift
func onDrag(_ data: @escaping () -> NSItemProvider) -> some View
```

Modiferになっていますので、ドラッグのプレビューが適用されたビューになります。

`onTapGesture(count:perform:)`と同じ理屈で、場合によって`contentShape(_:eoFill:)`で調整する必要があります。

```swift
view
    .padding()
    .contentShape(Rectangle()) // ないとパディングがドラッグのプレビューに反映されない
    .onDrag { ... }
```

Modiferに提供するのは`() -> NSItemProvider`のみで、クロージャーですので、どのビューがドラッグされるのを記録するのはちょうどいいように見えますが、ドラッグがキャンセルされた場合検知できませんので、記録できていてもリセットできません。現時点あくまで`NSItemProvider`の作成がlazyになっているだけです。

`NSItemProvider`の作成によく使われるイニシャライザは`init(object: NSItemProviderWriting)`で、`NSString`や`UIImage`などはすでに`NSItemProviderWriting`を実装済みですので、直接使うことが可能です。

自作のモデルを渡せたい場合、`NSItemProviderWriting`を実装する必要があります。すでに`Codable`を実装したタイプでは割と簡単です。

```swift
extension SomeCodable: NSItemProviderWriting {
    static var writableTypeIdentifiersForItemProvider: [String] {
        ["com.example.SomeCodable"]
    }

    func loadData(withTypeIdentifier typeIdentifier: String, forItemProviderCompletionHandler completionHandler: @escaping (Data?, Error?) -> Void) -> Progress? {
        do {
            let data = try JSONEncoder().encode(self)
            completionHandler(data, nil)
        } catch {
            completionHandler(nil, error)
        }
        return nil
    }
}
```

自作のtype identifierはuniform type identifier (UTI)の宣言を行う必要がありますので、`info.plist`に下記の内容を追加し、宣言します。

```xml
<key>UTExportedTypeDeclarations</key>
<array>
    <dict>
        <key>UTTypeIdentifier</key>
        <string>com.example.SomeCodable</string>
    </dict>
</array>
```

これでドラッグの準備ができました。

## ドロップ

ドロップのmodifierは下記の三つになります。

- ```swift
  func onDrop(of supportedContentTypes: [UTType], delegate: DropDelegate) -> some View
  ```

- ```swift
  func onDrop(of supportedContentTypes: [UTType], isTargeted: Binding<Bool>?, perform action: @escaping ([NSItemProvider]) -> Bool) -> some View`
  ```

- ```swift
  func onDrop(of supportedContentTypes: [UTType], isTargeted: Binding<Bool>?, perform action: @escaping ([NSItemProvider], CGPoint) -> Bool) -> some View`
  ```

`DropDelegate`の方はいろいろ制御できます[^1]が、`perform action`の方もドロップした座標が取れますので、普通の場合は十分です。

``` swift
.onDrop(of: [UTType(exportedAs: "com.example.SomeCodable")], // iOS 13は[String]
        isTargeted: nil) { itemProviders, location -> Bool in
    guard let itemProvider = itemProviders.first else {
        return false
    }
    itemProvider.loadObject(ofClass: SomeCodable.self) { item, error in
        guard let someCodable = item as? SomeCodable else {
            return
        }
        ...
    }
    return true
}
```

これでドロップできますが、`loadObject(ofClass:completionHandler:)`を使うには`SomeCodable`が`NSItemProviderReading`を実装する必要があります。

```swift
extension SomeCodable: NSItemProviderReading {

    static var readableTypeIdentifiersForItemProvider: [String] {
        ["com.example.SomeCodable"]
    }

    static func object(withItemProviderData data: Data, typeIdentifier: String) throws -> Self {
        try JSONDecoder().decode(Self.self, from: data)
    }
}
```

`NSItemProviderWriting`の逆ですね。

これにてドラッグ＆ドロップができるようになりました。

## 問題点

UIKitのドラッグ＆ドロップとの仕組みが違うようで、裏ではUIKitで描画するものにはドロップできない場合があります。私が気づいたのは下記になります。

- `TabView`の`tabItem(_:)`
- `List`（裏にある`UITableView`の`dragInteractionEnabled`を`true`にすれば一応ドラッグは可能だけど）

## まとめ

UIKitの場合よりできることは少ないですが、その割に実装は結構簡単です。

- ドラッグさせたいビューに`.onDrag`
- ドロップさせたいビューに`.onDrop`
- 自作モデルの場合`NSItemProviderWriting`と`NSItemProviderReading`を実装

## 参考

- [macos - Drag and drop with custom type identifier doesn't work - Stack Overflow](https://stackoverflow.com/questions/61225141/drag-and-drop-with-custom-type-identifier-doesnt-work)
- [Declaring New Uniform Type Identifiers](https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/understanding_utis/understand_utis_declare/understand_utis_declare.html#//apple_ref/doc/uid/TP40001319-CH204-SW1)

***

[^1]: 現時点`supportedContentTypes`と`validateDrop(info:)`は機能していません。