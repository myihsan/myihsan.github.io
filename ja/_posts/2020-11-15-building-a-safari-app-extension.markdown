---
layout: post
title: Safari機能拡張の作成
---

Big Surにアップデートし、Safariをデフォルトブラウザーにしてみたら、結構サクサクで気に入りましたが、愛用してた[Wikiwand](https://www.wikiwand.com)はSafariの機能拡張を提供していません。機能拡張とはいえ、ただWikipediaのURLを一定規則でWikiwandのURLに変更するものはずなので、自分で作りましょう！

## プロジェクト作成

macOS 10.14 からSafari機能拡張を提供するにはmacOSアプリを作成するしかないので、新規iOSプロジェクトを作成するようにXcodeから作成します。

「macOS」>「Safari Extension App」を選択し、次の画面にあるTypeをSafari App Extensionを選択し[^1]、作成します。

## 権限声明

テンプレートはすでに声明してくれましたが、必要な手順を確認しましょう。

`Info.plist`にある`NSExtension`の配下に、値が辞書型の`SFSafariWebsiteAccess`を追加します。

```xml
<key>SFSafariWebsiteAccess</key>
<dict>
    // ここで声明
</dict>
```

これからは権限の声明です。まずは権限レベルの声明です。

```xml
<key>Level</key>
<string>Some</string>
```

権限は`None`、`Some`、`All`で、今度はWikipediaのページをWikiwandにリダイレクトするので、`Some`で十分です。

`Some`ですので、対象ドメインを声明する必要があります。

```xml
<key>Allowed Domains</key>
<array>
    <string>*.wikipedia.org</string>
</array>
```

記事の言語により、サブドメインがかまりますので、ワイルドカードを使います。

ほとんどテンプレートに含まれ、実作業はドメインをWikipediaのドメインにしただけです。

## スクリプト追加

同様にテンプレートは追加してくれましたが、必要な手順を確認しましょう。

同じく`Info.plist`にある`NSExtension`の配下に声明する必要があります。

```xml
<key>SFSafariContentScript</key>
<array>
    <dict>
        <key>Script</key>
        <string>script.js</string>
    </dict>
</array>
```

配列の中に辞書型で声明します。多数のスクリプトを追加することが可能で、配列は必要ですが、なぜかスクリプトファイルを声明するには、キーが`Script`で、値をbundleに含まれたスクリプトファイルのパスにする必要があります。ちょっと間違いやすい形式ですよね。

声明が完了したら、残りはスクリプトファイルの追加と実装だけです。Safari機能拡張と本体アプリは別ターゲットで、Safari機能拡張にスクリプトファイルを追加する必要があります。

最後はリダイレクトの実装です。

```javascript
function redirectToWikiwand() {
    const pathnameComponents = location.pathname.split('/')
    const pathnameComponentsLength = pathnameComponents.length
    if ( pathnameComponents[pathnameComponentsLength - 2] != 'wiki' ) return

    const languageCode = location.hostname.split('.')[0]
    const title = pathnameComponents[pathnameComponentsLength - 1]
    if ( title == 'Main_Page' ) return

    const wikiwandURL = 'https://www.wikiwand.com/' + languageCode + '/' + title + location.hash
    location.replace(wikiwandURL)
}

redirectToWikiwand()
```

JavaScriptは詳しくないし、Wikiwand公式の変換方法もわからないので、もっと改善できるはずですが、これでWikiwandへリダイレクトできます。

## 使用

Apple Developer ProgramのアカウントがあればTeamを選び、Signing CertificateをDevelopmentにすれば、RunでSafari機能拡張を使えます[^2]。

Apple Developer Programのアカウントがない場合、Runしても「Safari」>「環境設定」>「機能拡張」にありませんが、「Safari」>「環境設定」>「メニューバーに"開発"メニューを表示」をチェックし、「開発」 > 「未署名の機能拡張を許可」しすれば、出てきます[^3]。

## 結論

JavaScriptの知識があれば、Safari機能拡張自体は難なく作成できます。もちろん複雑になると自作するのは大変になりますが、Safariをデフォルトブラウザとして使うため、欲しい機能拡張がまだMac App Storeにない場合、自分で作ってみましょう！

***

[^1]: Safari Web Extensionにすると、実装方法が変わります。

[^2]: 「Safari」>「環境設定」>「機能拡張」で有効にする必要があります。

[^3]: Safariを再起動したら、再設定する必要があります。
